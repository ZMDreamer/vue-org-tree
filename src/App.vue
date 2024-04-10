<template>
  <div class="w-full flex flex-col space-y-5">
    <div class="flex flex-col py-8 px-16 space-y-3 sticky top-0 shadow z-50 bg-white">
      <span class="text-3xl font-bold">Layout</span>
      <div class="inline-flex items-center space-x-3">
        <input
          v-model="layoutType"
          class="w-8 h-8"
          type="radio"
          name="layout"
          value="horizontal"
        />
        <span class="text-2xl">Horizontal</span>
      </div>
      <div class="inline-flex items-center space-x-3">
        <input
          v-model="layoutType"
          class="w-8 h-8"
          type="radio"
          name="layout"
          value="vertical"
        />
        <span class="text-2xl">Vertical</span>
      </div>
    </div>
    <div
      class="h-full w-full p-8 overflow-scroll scrolling-auto"
      style="position: relative;"
    >
      <vue2-org-tree
        v-draggable
        :data="data"
        :horizontal="layoutType == 'horizontal'"
        :collapsable="collapsable"
        :label-class-name="labelClassName"
        selected-class-name="bg-tomato"
        selected-key="selectedKey"
        @on-expand="onExpand"
        @on-node-click="onNodeClick"
      />
    </div>
  </div>
</template>

<script>
import Vue2OrgTree from "vue2-org-tree";

export default {
  name: "Appppp",
  components: {
    Vue2OrgTree,
  },
  directives: {
    draggable: {
      bind: function (el,) {

        el.style.cursor = 'move';
        el.addEventListener('mousedown', startDrag);
        function startDrag(event) {
          const startX = event.clientX;
          const startY = event.clientY;
          const initialX = el.offsetLeft;
          const initialY = el.offsetTop;

          function move(event) {
            const dx = event.clientX - startX;
            const dy = event.clientY - startY;
            const container = el.parentNode;
            const containerWidth = container.offsetWidth;
            const containerHeight = container.offsetHeight;
            const containerLeft = 0;
            const containerTop = 0;
            let left = initialX + dx;
            let top = initialY + dy;
            // 边界检查，确保元素不超出容器范围
            if (left < containerLeft) {
              left = containerLeft;
            } else if (left > containerLeft + containerWidth - el.offsetWidth) {
              left = containerLeft + containerWidth - el.offsetWidth;
            }

            if (top < containerTop) {
              top = containerTop;
            } else if (top > containerTop + containerHeight - el.offsetHeight) {
              top = containerTop + containerHeight - el.offsetHeight;
            }
            let bottom = containerHeight - top - el.offsetHeight;
            el.style.left = left + 'px';
            el.style.bottom = (bottom < 10 ? 10 : bottom) + 'px';
          }
          function stopDrag() {
            document.removeEventListener('mousemove', move);
            document.removeEventListener('mouseup', stopDrag);
          }
          document.addEventListener('mousemove', move);
          document.addEventListener('mouseup', stopDrag);
        }

      }
    }
  },
  data() {
    return {
      data: {},
      expandAll: true,
      layoutType: "horizontal",
      collapsable: true,
    };
  },
  mounted() {
    const data = require('../data.json');
    this.data = data.data[0]
    this.setData(this.data)
  },
  methods: {
    setData(data) {
      data.label = data.title
      if (data.children && data.children.length) {
        data.children.forEach(item => {
          this.setData(item);
        })
      }
    },
    labelClassName: function () {
      return "clickable-node";
    },

    onExpand: function (e, data) {
      console.log(e);
      if ("expand" in data) {
        data.expand = !data.expand;
        this.$set(data, "expand", data.expand);
        if (!data.expand && data.children) {
          this.collapse(data.children);
        }
      } else {
        this.$set(data, "expand", true);
      }
    },
    onNodeClick: function (e, data) {
      console.log("onNodeClick: %o", data);
      this.$set(data, "selectedKey", !data.selectedKey);
    },
    collapse: function (list) {
      var _this = this;
      list.forEach(function (child) {
        if (child.expand) {
          child.expand = false;
        }

        child.children && _this.collapse(child.children);
      });
    },
    expandChange: function () {
      this.toggleExpand(this.data, this.expandAll);
    },
    toggleExpand: function (data, val) {
      var _this = this;
      if (Array.isArray(data)) {
        data.forEach(function (item) {
          _this.$set(item, "expand", val);
          if (item.children) {
            _this.toggleExpand(item.children, val);
          }
        });
      } else {
        this.$set(data, "expand", val);
        if (data.children) {
          _this.toggleExpand(data.children, val);
        }
      }
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.h-full {
  height: 600px !important;
}
.w-full {
  width: 1000px !important;
}
</style>
