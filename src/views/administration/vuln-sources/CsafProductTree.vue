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
    
    <!-- Vulnerability Status Legend -->
    <div v-if="hasProductTree && hasVulnerabilityData" class="bg-secondary text-white mb-3 p-3 rounded">
      <small>
        <strong><i class="fa fa-info-circle"></i> Vulnerability Status:</strong>
        <b-badge variant="danger" class="ml-2">
          <i class="fa fa-exclamation-circle"></i> Known Affected
        </b-badge>
        <b-badge variant="warning" class="ml-2">
          <i class="fa fa-search"></i> Under Investigation
        </b-badge>
        <b-badge variant="success" class="ml-2">
          <i class="fa fa-check"></i> Fixed
        </b-badge>
        <b-badge variant="success" class="ml-2">
          <i class="fa fa-check-circle"></i> Not Affected
        </b-badge>
      </small>
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
        :product-vulnerability-status="productVulnerabilityStatus"
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
    productVulnerabilityStatus: {
      type: Object,
      default: () => ({}),
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
    hasVulnerabilityData() {
      return this.productVulnerabilityStatus && Object.keys(this.productVulnerabilityStatus).length > 0;
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
        productVulnerabilityStatus: {
          type: Object,
          default: () => ({}),
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
        productId() {
          return this.node.product?.product_id || null;
        },
        vulnerabilityInfo() {
          if (!this.productId || !this.productVulnerabilityStatus[this.productId]) {
            return null;
          }
          return this.productVulnerabilityStatus[this.productId];
        },
        vulnerabilityStatusBadges() {
          if (!this.vulnerabilityInfo) return [];
          
          const badges = [];
          const statuses = this.vulnerabilityInfo.statuses;
          
          // Priority order: show most critical status first
          if (statuses.has('known_affected')) {
            badges.push({ 
              variant: 'danger', 
              icon: 'fa-exclamation-circle', 
              text: 'Known Affected',
              count: this.vulnerabilityInfo.vulnerabilities.filter(v => v.status === 'known_affected').length
            });
          }
          if (statuses.has('under_investigation')) {
            badges.push({ 
              variant: 'warning', 
              icon: 'fa-search', 
              text: 'Under Investigation',
              count: this.vulnerabilityInfo.vulnerabilities.filter(v => v.status === 'under_investigation').length
            });
          }
          if (statuses.has('fixed')) {
            badges.push({ 
              variant: 'success', 
              icon: 'fa-check', 
              text: 'Fixed',
              count: this.vulnerabilityInfo.vulnerabilities.filter(v => v.status === 'fixed').length
            });
          }
          if (statuses.has('known_not_affected')) {
            badges.push({ 
              variant: 'success', 
              icon: 'fa-check-circle', 
              text: 'Not Affected',
              count: this.vulnerabilityInfo.vulnerabilities.filter(v => v.status === 'known_not_affected').length
            });
          }
          
          return badges;
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
        getVulnerabilityTooltip(badge) {
          if (!this.vulnerabilityInfo) return '';
          
          const vulns = this.vulnerabilityInfo.vulnerabilities.filter(v => {
            const statusKey = v.status.toLowerCase().replace(/_/g, '_');
            const badgeText = badge.text.toLowerCase().replace(/ /g, '_');
            return statusKey === badgeText || v.status === badgeText;
          });
          
          let tooltip = `<strong>${badge.text}</strong><br/>`;
          tooltip += `<small>${badge.count} vulnerability(ies):</small><br/>`;
          vulns.forEach(v => {
            tooltip += `• ${v.id}${v.title ? ': ' + v.title : ''}<br/>`;
          });
          
          return tooltip;
        },
      },
      template: `
        <div :style="{ marginLeft: (level * 1.5) + 'rem' }" class="my-2">
          <div class="p-3 bg-dark text-white border rounded cursor-pointer" @click="toggle">
            <div class="d-flex align-items-center flex-wrap">
                <span class="node-expand-icon mx-2" v-if="hasChildren">
                    <i :class="isExpanded ? 'fa fa-chevron-down' : 'fa fa-chevron-right'"></i>
                </span>
                <span class="node-expand-icon placeholder" v-else></span>
                <i class="fa text-light mr-2" :class="categoryIcon"></i>
                <b-badge 
                  variant="light" 
                  class="mr-2"
                  v-if="node.category"
                >
                  {{ fmt(node.category) }}
                </b-badge>
                <strong class="text-white" v-if="node.name">{{ fmt(node.name) }}</strong>
                
                <!-- Vulnerability Status Badges -->
                <span v-if="vulnerabilityStatusBadges.length > 0" class="vulnerability-badges ml-2">
                  <b-badge 
                    v-for="(badge, idx) in vulnerabilityStatusBadges" 
                    :key="idx"
                    :variant="badge.variant"
                    class="ml-1"
                    v-b-tooltip.hover.html 
                    :title="getVulnerabilityTooltip(badge)"
                  >
                    <i class="fa" :class="badge.icon"></i> {{ badge.count }}
                  </b-badge>
                </span>
            </div>
            
            <div v-if="node.product" class="mt-2">
              <small class="text-light">
                <i class="fa fa-info-circle"></i>
                <span v-if="node.product.product_id">
                  ID: <code class="bg-secondary text-white px-2 py-1 rounded">{{ node.product.product_id }}</code>
                </span>
                <span v-if="node.product.name && node.product.name !== node.name">
                  | {{ node.product.name }}
                </span>
              </small>
            </div>
          </div>
          
          <div v-if="hasChildren && isExpanded" class="mt-2">
            <product-node 
              v-for="(child, i) in node.branches" 
              :key="i"
              :index="i"
              :node="child"
              :level="level + 1"
              :expanded-nodes="expandedNodes"
              :product-vulnerability-status="productVulnerabilityStatus"
              @toggle="handleChildToggle"
            />
          </div>
        </div>
      `,
    },
  },
};
</script>

<style scoped>
/* Minimal custom CSS - rely on Bootstrap */
.node-expand-icon {
  width: 20px;
  min-width: 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-right: 0.5rem;
}

.node-expand-icon.placeholder {
  opacity: 0;
}

.cursor-pointer {
  cursor: pointer;
}

/* Subtle pulse animation for critical vulnerabilities */
.badge-danger {
  animation: pulse-danger 2s infinite;
}

@keyframes pulse-danger {
  0%, 100% {
    box-shadow: 0 0 0 0 rgba(220, 53, 69, 0.4);
  }
  50% {
    box-shadow: 0 0 0 4px rgba(220, 53, 69, 0);
  }
}
</style>
