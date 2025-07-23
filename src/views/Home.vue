<template>
  <a-layout class="layout">
    <a-layout-header class="layout-header">
      <div class="logo">工单列表</div>
      <a-menu mode="horizontal" class="menu-horizontal">
        <a-menu-item key="user">欢迎回来({{ userRole }})</a-menu-item>
        <a-menu-item key="logout" @click="handleLogout">退出登录</a-menu-item>
      </a-menu>
    </a-layout-header>
    <a-layout-content class="layout-content">
      <div class="content-container">
        <div class="table-container">
          <a-table :columns="columns" :data-source="dataSource" row-key="id" />
        </div>
        <div class="chart-container" id="projectHoursChart"></div>
      </div>
    </a-layout-content>
  </a-layout>
</template>

<script setup>
import { useRouter } from "vue-router";
import { ref, h, onMounted, watch, onUnmounted } from "vue";
import * as echarts from "echarts";
import { Button as aButton, message } from "ant-design-vue";
import { projects } from "../mock.js";

const router = useRouter();

// 获取用户角色
const userRole = ref(localStorage.getItem("userRole") || "user");

const handleLogout = () => {
  router.push("/login");
};

const dataSource = ref(projects);
let chartInstance;

const updateChart = () => {
  // 聚合项目工时数据
  const projectHours = {};
  dataSource.value.forEach((item) => {
    if (!projectHours[item.project]) {
      projectHours[item.project] = 0;
    }
    projectHours[item.project] += Number(item.hours);
  });

  // 准备图表数据
  const projectNames = Object.keys(projectHours);
  const hoursData = Object.values(projectHours);

  // 初始化图表
  if (!chartInstance) {
    chartInstance = echarts.init(document.getElementById("projectHoursChart"));
  }

  // 设置图表选项
  const option = {
    title: {
      text: "Project Hours Distribution",
      left: "center",
    },
    tooltip: {
      trigger: "axis",
      axisPointer: {
        type: "shadow",
      },
    },
    grid: {
      left: "3%",
      right: "4%",
      bottom: "3%",
      containLabel: true,
    },
    xAxis: {
      type: "category",
      data: projectNames,
      axisLabel: {
        interval: 0,
      },
    },
    yAxis: {
      type: "value",
    },
    series: [
      {
        name: "Total Hours",
        type: "bar",
        data: hoursData,
        itemStyle: {
          color: "#1890ff",
        },
      },
    ],
  };

  chartInstance.setOption(option);
};

// 初始化图表
onMounted(() => {
  updateChart();
  // 监听窗口大小变化，调整图表
  window.addEventListener("resize", () => {
    chartInstance?.resize();
  });
});

// 监听数据变化，更新图表
watch(dataSource, () => {
  updateChart();
});

// 组件卸载时销毁图表
onUnmounted(() => {
  chartInstance?.dispose();
});

const columns = [
  {
    title: "ID",
    dataIndex: "id",
    key: "id",
  },
  {
    title: "Project",
    dataIndex: "project",
    key: "project",
  },
  {
    title: "Overtime",
    dataIndex: "overtime",
    key: "overtime",
    customRender: ({ text }) => (text == true ? "Yes" : "No"),
  },
  {
    title: "Hours",
    dataIndex: "hours",
    key: "hours",
  },
  {
    title: "Created At",
    dataIndex: "created_at",
    key: "created_at",
  },
  {
    title: "操作",
    key: "action",
    width: 120,
    customRender: ({ text, record }) => {
      if (!record || !record.id) {
        return h("span", "Invalid Record");
      }

      return h(
        aButton,
        {
          type: "primary",
          danger: true,
          onClick: () => handleDelete(record.id),
          disabled: userRole.value !== "admin",
        },
        () => "Delete"
      );
    },
  },
];

const handleDelete = (id) => {
  message.success("删除成功");
  dataSource.value = dataSource.value.filter((project) => project.id !== id);
};
</script>

<style lang="less" scoped>
.layout-header {
  background: #fff;
  padding: 0 20px;

  .logo {
    float: left;
  }

  .menu-horizontal {
    line-height: 64px;
    float: right;
  }
}

.layout-content {
  padding: 24px;
  min-height: 280px;

  .content-container {
    display: flex;
    gap: 24px;
    margin-bottom: 24px;

    .table-container {
      background: #fff;
      padding: 24px;
      min-height: 280px;
      flex: 1;
      min-width: 500px;
    }

    .chart-container {
      height: 400px;
      flex: 1;
      min-width: 500px;
    }
  }
}

.layout-footer {
  text-align: center;
}
</style>