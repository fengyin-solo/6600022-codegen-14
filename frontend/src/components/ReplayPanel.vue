<template>
  <div class="bg-gray-900 rounded-xl p-4 border border-gray-700">
    <div class="flex items-center justify-between mb-3">
      <h3 class="text-lg font-bold text-green-400">棋谱回放</h3>
      <button
        @click="showGroupManager = !showGroupManager"
        class="text-xs px-2 py-1 bg-gray-800 text-gray-400 rounded hover:bg-gray-700 hover:text-gray-200 transition-colors"
      >
        {{ showGroupManager ? '完成' : '管理分组' }}
      </button>
    </div>

    <!-- Group Filter Tabs -->
    <div v-if="store.status !== 'replaying'" class="mb-3">
      <div class="flex flex-wrap gap-1.5">
        <button
          @click="store.setFilterGroup(null)"
          class="px-3 py-1 text-xs rounded-full transition-colors"
          :class="!store.currentFilterGroupId ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          全部 ({{ store.gameRecords.length }})
        </button>
        <button
          v-for="group in store.favoriteGroups"
          :key="group.id"
          @click="store.setFilterGroup(group.id)"
          class="px-3 py-1 text-xs rounded-full transition-colors flex items-center gap-1"
          :class="store.currentFilterGroupId === group.id ? 'bg-yellow-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          <svg v-if="!showGroupManager" class="w-3 h-3" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
          </svg>
          <span>{{ group.name }}</span>
          <span class="opacity-70">({{ store.getRecordCountInGroup(group.id) }})</span>
          <template v-if="showGroupManager">
            <button
              @click.stop="startRenameGroup(group)"
              class="ml-1 text-gray-400 hover:text-white"
              title="重命名"
            >
              <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/>
              </svg>
            </button>
            <button
              @click.stop="handleDeleteGroup(group.id)"
              class="text-red-400 hover:text-red-300"
              title="删除"
            >
              <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
              </svg>
            </button>
          </template>
        </button>
      </div>
      <button
        v-if="showGroupManager"
        @click="showNewGroupInput = true; newGroupName = ''"
        class="mt-2 w-full py-1.5 border border-dashed border-gray-600 text-gray-500 text-xs rounded-lg hover:border-green-500 hover:text-green-400 transition-colors"
      >
        + 新建分组
      </button>
    </div>

    <!-- New Group Input -->
    <div v-if="showNewGroupInput" class="mb-3 p-2 bg-gray-800 rounded-lg">
      <input
        v-model="newGroupName"
        @keyup.enter="handleCreateGroup"
        type="text"
        placeholder="输入分组名称"
        class="w-full px-2 py-1 bg-gray-900 border border-gray-600 rounded text-sm text-white placeholder-gray-500 focus:outline-none focus:border-green-500"
        ref="newGroupInput"
      />
      <div class="flex gap-2 mt-2">
        <button
          @click="handleCreateGroup"
          class="flex-1 py-1 bg-green-600 text-white text-xs rounded hover:bg-green-500 transition-colors"
        >
          创建
        </button>
        <button
          @click="showNewGroupInput = false"
          class="flex-1 py-1 bg-gray-700 text-gray-300 text-xs rounded hover:bg-gray-600 transition-colors"
        >
          取消
        </button>
      </div>
    </div>

    <!-- Rename Group Input -->
    <div v-if="renamingGroupId" class="mb-3 p-2 bg-gray-800 rounded-lg">
      <input
        v-model="renameGroupName"
        @keyup.enter="handleRenameGroup"
        type="text"
        placeholder="输入新名称"
        class="w-full px-2 py-1 bg-gray-900 border border-gray-600 rounded text-sm text-white placeholder-gray-500 focus:outline-none focus:border-green-500"
        ref="renameGroupInput"
      />
      <div class="flex gap-2 mt-2">
        <button
          @click="handleRenameGroup"
          class="flex-1 py-1 bg-green-600 text-white text-xs rounded hover:bg-green-500 transition-colors"
        >
          确定
        </button>
        <button
          @click="renamingGroupId = null"
          class="flex-1 py-1 bg-gray-700 text-gray-300 text-xs rounded hover:bg-gray-600 transition-colors"
        >
          取消
        </button>
      </div>
    </div>

    <!-- Record List -->
    <div v-if="store.status !== 'replaying'">
      <p v-if="store.filteredRecords.length === 0" class="text-gray-500 text-sm text-center py-6">
        {{ store.currentFilterGroupId ? '该分组暂无棋谱' : '暂无棋谱记录' }}
      </p>
      <div v-else class="space-y-2 max-h-64 overflow-y-auto pr-1">
        <div
          v-for="record in store.filteredRecords"
          :key="record.id"
          class="bg-gray-800 rounded-lg p-3 cursor-pointer hover:bg-gray-700 transition-colors border border-gray-700 group"
        >
          <div class="flex items-start justify-between" @click="store.startReplay(record)">
            <div class="flex-1 min-w-0">
              <div class="flex items-center gap-2">
                <span class="text-sm text-gray-300">{{ record.createdAt }}</span>
                <span
                  class="text-xs px-2 py-0.5 rounded-full"
                  :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
                >
                  {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
                </span>
              </div>
              <div class="text-xs text-gray-500 mt-1">共 {{ record.moves.length }} 手</div>
              <div v-if="store.getGroupsForRecord(record.id).length > 0" class="flex flex-wrap gap-1 mt-1.5">
                <span
                  v-for="g in store.getGroupsForRecord(record.id)"
                  :key="g.id"
                  class="text-xs px-1.5 py-0.5 bg-yellow-900/30 text-yellow-400 rounded"
                >
                  {{ g.name }}
                </span>
              </div>
            </div>
          </div>
          <div class="flex items-center justify-end mt-2 gap-1">
            <button
              @click.stop="openFavoriteDialog(record)"
              class="text-xs px-2 py-1 rounded transition-colors flex items-center gap-1"
              :class="isFavorited(record.id) ? 'bg-yellow-600/20 text-yellow-400 border border-yellow-600/50' : 'bg-gray-700 text-gray-400 hover:bg-gray-600 hover:text-gray-200'"
              :title="isFavorited(record.id) ? '已收藏' : '添加收藏'"
            >
              <svg class="w-3.5 h-3.5" :fill="isFavorited(record.id) ? 'currentColor' : 'none'" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z"/>
              </svg>
              <span>收藏</span>
            </button>
            <button
              @click.stop="deleteRecord(record.id)"
              class="text-xs px-2 py-1 rounded bg-gray-700 text-gray-400 hover:bg-red-600/20 hover:text-red-400 transition-colors opacity-0 group-hover:opacity-100"
              title="删除棋谱"
            >
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Favorite Dialog -->
    <div
      v-if="favoriteDialogVisible"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
      @click.self="favoriteDialogVisible = false"
    >
      <div class="bg-gray-900 rounded-xl p-5 border border-gray-700 w-80 max-w-[90vw]">
        <h4 class="text-lg font-bold text-green-400 mb-3">添加到收藏夹</h4>
        <div v-if="store.favoriteGroups.length === 0" class="text-gray-500 text-sm text-center py-4">
          暂无分组，请先创建分组
        </div>
        <div v-else class="space-y-2 max-h-60 overflow-y-auto">
          <label
            v-for="group in store.favoriteGroups"
            :key="group.id"
            class="flex items-center gap-3 p-2 rounded-lg cursor-pointer hover:bg-gray-800 transition-colors"
          >
            <input
              type="checkbox"
              :checked="store.isRecordInGroup(selectedRecordId!, group.id)"
              @change="toggleFavorite(group.id)"
              class="w-4 h-4 accent-green-500"
            />
            <span class="text-gray-200 text-sm">{{ group.name }}</span>
            <span class="text-xs text-gray-500">({{ store.getRecordCountInGroup(group.id) }})</span>
          </label>
        </div>
        <div class="mt-4 flex gap-2">
          <button
            @click="favoriteDialogVisible = false"
            class="flex-1 py-2 bg-gray-800 text-gray-300 rounded-lg hover:bg-gray-700 transition-colors text-sm"
          >
            关闭
          </button>
        </div>
      </div>
    </div>

    <!-- Replay Controls -->
    <div v-else>
      <div class="text-center mb-3">
        <span class="text-gray-400 text-sm">第 {{ store.replayIndex }} / {{ store.replayMoves.length }} 手</span>
      </div>

      <div class="w-full bg-gray-800 rounded-full h-2 mb-4">
        <div
          class="bg-green-500 h-2 rounded-full transition-all"
          :style="{ width: `${(store.replayIndex / store.replayMoves.length) * 100}%` }"
        />
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <button @click="store.replayGoToStart()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="回到开始">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"/></svg>
        </button>
        <button @click="store.replayStepBack()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="上一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="store.toggleReplayPlay()" class="p-3 bg-green-600 rounded-lg hover:bg-green-500 text-white transition-colors" :title="store.isReplayPlaying ? '暂停' : '播放'">
          <svg v-if="!store.isReplayPlaying" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
          <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 4h4v16H6zM14 4h4v16h-4z"/></svg>
        </button>
        <button @click="store.replayStepForward()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="下一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
        <button @click="store.replayGoToEnd()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="跳到结尾">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"/></svg>
        </button>
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <span class="text-xs text-gray-500">速度:</span>
        <button
          v-for="speed in speeds"
          :key="speed.value"
          @click="store.setReplaySpeed(speed.value)"
          class="px-2 py-1 text-xs rounded transition-colors"
          :class="store.replaySpeed === speed.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ speed.label }}
        </button>
      </div>

      <button
        @click="store.stopReplay()"
        class="w-full py-2 bg-red-600/20 border border-red-600/50 text-red-400 rounded-lg hover:bg-red-600/30 transition-colors text-sm"
      >
        退出回放
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';
import { useGameStore } from '../store/game';

const store = useGameStore();

const speeds = [
  { label: '慢', value: 2000 },
  { label: '中', value: 1000 },
  { label: '快', value: 500 },
  { label: '极快', value: 200 },
];

const showGroupManager = ref(false);
const showNewGroupInput = ref(false);
const newGroupName = ref('');
const newGroupInput = ref<HTMLInputElement | null>(null);

const renamingGroupId = ref<string | null>(null);
const renameGroupName = ref('');
const renameGroupInput = ref<HTMLInputElement | null>(null);

const favoriteDialogVisible = ref(false);
const selectedRecordId = ref<string | null>(null);

function isFavorited(recordId: string): boolean {
  return store.favoriteGroups.some(g => store.isRecordInGroup(recordId, g.id));
}

function openFavoriteDialog(record: { id: string }) {
  selectedRecordId.value = record.id;
  favoriteDialogVisible.value = true;
}

function toggleFavorite(groupId: string) {
  if (!selectedRecordId.value) return;
  if (store.isRecordInGroup(selectedRecordId.value, groupId)) {
    store.removeFromFavorites(selectedRecordId.value, groupId);
  } else {
    store.addToFavorites(selectedRecordId.value, groupId);
  }
}

function handleCreateGroup() {
  if (!newGroupName.value.trim()) return;
  store.createFavoriteGroup(newGroupName.value.trim());
  showNewGroupInput.value = false;
  newGroupName.value = '';
}

function startRenameGroup(group: { id: string; name: string }) {
  renamingGroupId.value = group.id;
  renameGroupName.value = group.name;
  nextTick(() => {
    renameGroupInput.value?.focus();
    renameGroupInput.value?.select();
  });
}

function handleRenameGroup() {
  if (!renamingGroupId.value || !renameGroupName.value.trim()) return;
  store.renameFavoriteGroup(renamingGroupId.value, renameGroupName.value.trim());
  renamingGroupId.value = null;
  renameGroupName.value = '';
}

function handleDeleteGroup(groupId: string) {
  if (confirm('确定要删除这个分组吗？分组内的收藏关系也会被移除。')) {
    store.deleteFavoriteGroup(groupId);
  }
}

function deleteRecord(recordId: string) {
  if (confirm('确定要删除这条棋谱吗？')) {
    store.gameRecords = store.gameRecords.filter(r => r.id !== recordId);
    store.favoriteItems = store.favoriteItems.filter(item => item.recordId !== recordId);
  }
}
</script>
