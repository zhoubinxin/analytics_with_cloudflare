# 基于 Cloudflare + Huno + D1 的网页访客统计服务

[原作者演示站](https://webviso.yestool.org/)

## 部署步骤

### 安装依赖

```
npm install -g wrangler
npm install hono
```

### 创建D1数据库：[web_analytics]

> 数据库名称为`web_analytics`，与`package.json`内保持一致

```
npx wrangler d1 create web_analytics
```

运行后控制台显示
```
✅ Successfully created DB 'web_analytics' in region APAC
Created your new D1 database.

[[d1_databases]]
binding = "DB" # i.e. available in your Worker on env.DB
database_name = "web_analytics"
database_id = "xxx"
```
修改 wrangler.toml 中的 database_id

### 初始化D1数据库的表结构

```
npm run initSql
```

### 绑定自定义域名

```
routes = [{ pattern = "example.com", custom_domain = true }]
```

### 发布

```
npm run deploy
```

## 使用

```vue
<template>
  本页访问人次:<span>{{webviso.uv}}</span><br>
  本页访问人数:<span>{{ webviso.pv }}</span>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const webvisoLoaded = ref(false);
const webviso = ref({
  uv: '',
  pv: '',
});

const loadWebviso = async () => {
  const apiUrl = 'https://analytics.xbxin.com/api/visit'

  const requestData = {
    url: window.location.pathname,
    hostname: window.location.hostname,
    referrer: document.referrer,
    pv: true,
    uv: true,
  }

  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(requestData),
    })

    if (response.ok) {
      const data = await response.json();
      webviso.value = data.data;
    } else {
      console.error('Error loading analytics data:', response.statusText);
    }
  } catch (err) {
    console.error('Error loading analytics script:', err);
  }
}

onMounted(() => {
  loadWebviso();
});
</script>
```
