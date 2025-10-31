<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  level: {
    type: String,
    required: true,
    validator: (value) => ['S', 'A'].includes(value) // 仅保留S/A级
  }
});

const ratedProjects = ref([]);
const loading = ref(true);

const loadRatingData = async () => {
  try {
    const basePath = import.meta.env.BASE_URL || '/';
    const dataPath = `${basePath}data/datawhalechina/project_ratings.json`.replace(/\/+/g, '/');
    const response = await fetch(dataPath);
    
    if (!response.ok) throw new Error('数据加载失败');
    
    const data = await response.json();
    // 仅按等级筛选，无需排序（无star_count字段）
    ratedProjects.value = data.projectRatings.filter(
      project => project.level === props.level
    );
  } catch (error) {
    console.error('加载项目评级数据失败:', error);
  } finally {
    loading.value = false;
  }
};

onMounted(loadRatingData);
</script>

<template>
  <div class="project-ratings-container">
    <div v-if="loading" class="loading-state">
      <p>加载中...</p>
    </div>
    
    <div v-else-if="ratedProjects.length === 0" class="empty-state">
      <p>暂无{{ props.level }}级项目</p>
    </div>
    
    <div v-else class="projects-list">
      <div class="project-card leaderboard-item-base" v-for="(project, index) in ratedProjects" :key="project.name">
        <div class="project-info">
          <h3 class="project-name">
            <a :href="project.url" target="_blank" rel="noopener">{{ project.name }}</a>
            <span class="level-badge">{{ props.level }}级</span>
          </h3>
          <p class="project-desc">{{ project.msg }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.projects-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-top: 16px;
}

.project-card {
  padding: 16px;
  border-radius: 8px;
  background: var(--vp-c-bg-soft);
}

.project-name {
  margin: 0 0 8px 0;
  font-size: 18px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.project-name a {
  color: var(--vp-c-brand-1);
  text-decoration: none;
}

.project-name a:hover {
  text-decoration: underline;
}

.level-badge {
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 12px;
  background: var(--vp-c-brand-soft);
  color: var(--vp-c-brand-1);
  font-weight: 600;
}

.project-desc {
  margin: 0;
  font-size: 14px;
  color: var(--vp-c-text-2);
  line-height: 1.6;
}

.loading-state, .empty-state {
  padding: 40px 0;
  text-align: center;
  color: var(--vp-c-text-2);
}
</style>
