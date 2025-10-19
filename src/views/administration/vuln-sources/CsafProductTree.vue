<template>
  <div class="csaf-product-tree">
    <div v-if="hasProductTree" class="d-flex align-items-center justify-content-between mb-3">
      <h5 class="mb-0">Product Tree</h5>
      <div class="tree-controls">
        <b-button
          size="sm"
          variant="outline-secondary"
          @click="expandAll"
          class="mr-2"
        >
          <i class="fa fa-plus-square-o"></i> Expand All
        </b-button>
        <b-button
          size="sm"
          variant="outline-secondary"
          @click="collapseAll"
        >
          <i class="fa fa-minus-square-o"></i> Collapse All
        </b-button>
      </div>
    </div>
    <div v-if="!hasProductTree" class="text-muted">
      <i class="fa fa-info-circle"></i> No product_tree found in this CSAF document.
    </div>
    <div v-if="hasProductTree" class="product-tree-container">
      <product-node
        v-for="(node, idx) in rootNodes"
        :key="idx"
        :node="node"
        :level="0"
        :expanded-nodes="expandedNodes"
        @toggle="toggleNode"
      />
    </div>
  </div>
</template>

<script>
// A modern recursive renderer for CSAF product_tree structures.
// Features: expand/collapse, visual hierarchy, category badges, tooltips

export default {
  name: 'CsafProductTree',
  props: {
    content: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      expandedNodes: new Set(),
    };
  },
  computed: {
    hasProductTree() {
      if (!this.content) return false;
      const pt = this.content.product_tree || this.content.document?.product_tree;
      return !!pt;
    },
    rootNodes() {
      if (!this.content) return [];
      const pt = this.content.product_tree || this.content.document?.product_tree;
      if (!pt) return [];
      if (Array.isArray(pt)) return pt;
      if (pt.branches && Array.isArray(pt.branches)) return pt.branches;
      return [pt];
    },
  },
  methods: {
    toggleNode(nodeId) {
      if (this.expandedNodes.has(nodeId)) {
        this.expandedNodes.delete(nodeId);
      } else {
        this.expandedNodes.add(nodeId);
      }
      // Force reactivity
      this.expandedNodes = new Set(this.expandedNodes);
    },
    expandAll() {
      const allIds = new Set();
      const collectIds = (nodes) => {
        if (!Array.isArray(nodes)) return;
        nodes.forEach((node, idx) => {
          const id = this.getNodeId(node, idx);
          allIds.add(id);
          if (node.branches && Array.isArray(node.branches)) {
            collectIds(node.branches);
          }
        });
      };
      collectIds(this.rootNodes);
      this.expandedNodes = allIds;
    },
    collapseAll() {
      this.expandedNodes = new Set();
    },
    getNodeId(node, index) {
      return `${node.category || 'unknown'}-${node.name || ''}-${index}`;
    },
  },
  components: {
    ProductNode: {
      name: 'ProductNode',
      props: {
        node: {
          type: Object,
          required: true,
        },
        level: {
          type: Number,
          default: 0,
        },
        expandedNodes: {
          type: Set,
          required: true,
        },
        index: {
          type: Number,
          default: 0,
        },
      },
      computed: {
        nodeId() {
          return `${this.node.category || 'unknown'}-${this.node.name || ''}-${this.index}`;
        },
        isExpanded() {
          return this.expandedNodes.has(this.nodeId);
        },
        hasChildren() {
          return this.node.branches && Array.isArray(this.node.branches) && this.node.branches.length > 0;
        },
        categoryVariant() {
          const category = (this.node.category || '').toLowerCase();
          const variants = {
            'vendor': 'primary',
            'product_family': 'info',
            'product_name': 'success',
            'product_version': 'warning',
            'architecture': 'secondary',
            'host_name': 'dark',
            'language': 'light',
            'legacy': 'danger',
            'patch_level': 'info',
            'service_pack': 'warning',
            'specification': 'secondary',
          };
          return variants[category] || 'secondary';
        },
        categoryIcon() {
          const category = (this.node.category || '').toLowerCase();
          const icons = {
            'vendor': 'fa-building',
            'product_family': 'fa-cubes',
            'product_name': 'fa-cube',
            'product_version': 'fa-tag',
            'architecture': 'fa-microchip',
            'host_name': 'fa-server',
            'language': 'fa-language',
            'legacy': 'fa-history',
            'patch_level': 'fa-wrench',
            'service_pack': 'fa-archive',
            'specification': 'fa-file-text',
          };
          return icons[category] || 'fa-folder';
        },
      },
      methods: {
        fmt(v) {
          if (v === null || v === undefined) return '';
          if (Array.isArray(v)) return v.join(' / ');
          if (typeof v === 'object') {
            try {
              return JSON.stringify(v);
            } catch (e) {
              return String(v);
            }
          }
          return String(v);
        },
        toggle() {
          if (this.hasChildren) {
            this.$emit('toggle', this.nodeId);
          }
        },
        handleChildToggle(childNodeId) {
          this.$emit('toggle', childNodeId);
        },
      },
      template: `
        <div class="product-node" :class="'level-' + level" :style="{ marginLeft: (level * 1.5) + 'rem' }">
          <div class="node-content" @click="toggle">
            
                        
            <div class="node-info">
              <div class="node-header">
                <span class="node-expand-icon" v-if="hasChildren">
                    <i :class="isExpanded ? 'fa fa-chevron-down' : 'fa fa-chevron-right'"></i>
                </span>
                <span class="node-expand-icon placeholder" v-else></span>
                <i class="fa" :class="categoryIcon"></i>
                <b-badge 
                  :variant="categoryVariant" 
                  class="category-badge"
                  v-if="node.category"
                >
                  {{ fmt(node.category) }}
                </b-badge>
                <span class="node-name" v-if="node.name">{{ fmt(node.name) }}</span>
              </div>
              
              <div class="node-metadata" v-if="node.product">
                <small class="text-muted">
                  <i class="fa fa-info-circle"></i>
                  <span v-if="node.product.product_id">
                    ID: <code>{{ node.product.product_id }}</code>
                  </span>
                  <span v-if="node.product.name && node.product.name !== node.name">
                    | {{ node.product.name }}
                  </span>
                </small>
              </div>
            </div>
          </div>
          
          <transition name="slide-fade">
            <div v-if="hasChildren && isExpanded" class="node-children">
              <product-node 
                v-for="(child, i) in node.branches" 
                :key="i"
                :index="i"
                :node="child"
                :level="level + 1"
                :expanded-nodes="expandedNodes"
                @toggle="handleChildToggle"
              />
            </div>
          </transition>
        </div>
      `,
    },
  },
};
</script>

<style scoped>
.csaf-product-tree {
  margin-top: 1rem;
}

.tree-controls {
  display: flex;
  gap: 0.5rem;
}

.product-tree-container {
  background: transparent;
  border-radius: 8px;
  padding: 0;
  border: none;
}

.product-node {
  margin: 0.75rem 0;
  position: relative;
}

.node-content {
  display: flex;
  align-items: center;
  padding: 0.85rem 1rem;
  background: rgba(0, 0, 0, 0.02);
  border-radius: 6px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.node-content:hover {
  background: rgba(0, 123, 255, 0.05);
  border-color: rgba(0, 123, 255, 0.3);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.12);
  transform: translateX(2px);
}

.node-expand-icon {
  width: 20px;
  min-width: 20px;
  height: 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-right: 0.5rem;
  color: #6c757d;
  font-size: 0.85rem;
  transition: transform 0.2s ease;
  flex-shrink: 0;
}

.node-expand-icon:hover {
  color: #007bff;
}

.node-expand-icon.placeholder {
  opacity: 0;
  pointer-events: none;
}

.node-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  align-self: center;
}

.node-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.node-header > .fa {
  color: #6c757d;
  font-size: 1rem;
}

.category-badge {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  padding: 0.25rem 0.5rem;
}

.node-name {
  font-weight: 500;
  color: #212529;
  font-size: 0.95rem;
}

.node-metadata {
  margin-top: 0.5rem;
  padding-left: 0;
}

.node-metadata code {
  background: #e9ecef;
  padding: 0.1rem 0.3rem;
  border-radius: 3px;
  font-size: 0.85rem;
  color: #495057;
}

.node-children {
  margin-top: 0.75rem;
  margin-left: 0;
  padding-left: 0;
  border-left: 2px solid rgba(0, 123, 255, 0.2);
  position: relative;
}

/* Level-based styling for depth indication */
.product-node.level-0 .node-content {
  border-left: 4px solid #007bff;
}

.product-node.level-1 .node-content {
  border-left: 4px solid #17a2b8;
}

.product-node.level-2 .node-content {
  border-left: 4px solid #28a745;
}

.product-node.level-3 .node-content {
  border-left: 4px solid #ffc107;
}

.product-node.level-4 .node-content {
  border-left: 4px solid #fd7e14;
}

.product-node.level-5 .node-content,
.product-node.level-6 .node-content,
.product-node.level-7 .node-content {
  border-left: 4px solid #6c757d;
}

/* Smooth transitions */
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
  transition: all 0.2s ease-in;
}

.slide-fade-enter {
  transform: translateY(-10px);
  opacity: 0;
}

.slide-fade-leave-to {
  transform: translateY(-5px);
  opacity: 0;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .tree-controls {
    flex-direction: column;
  }
  
  .tree-controls button {
    width: 100%;
  }
  
  .node-children {
    margin-left: 1rem;
    padding-left: 0.5rem;
  }
  
  .node-header {
    flex-direction: column;
    align-items: flex-start;
  }
}

/* Dark mode support (optional - if your app supports it) */
@media (prefers-color-scheme: dark) {
  .node-content {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.1);
    color: #e2e8f0;
  }
  
  .node-content:hover {
    background: rgba(0, 123, 255, 0.1);
    border-color: rgba(0, 123, 255, 0.3);
  }
  
  .node-name {
    color: #e2e8f0;
  }
  
  .node-metadata code {
    background: rgba(255, 255, 255, 0.1);
    color: #e2e8f0;
  }
  
  .node-children {
    border-left-color: rgba(0, 123, 255, 0.3);
  }
}
</style>
