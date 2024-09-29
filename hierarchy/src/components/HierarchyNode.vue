<template>
    <div class="node">
      <div class="card" :style="{ borderColor: getDepartmentColor(node.department) }">
        <div class="avatar-circle" :style="{ borderColor: getDepartmentColor(node.department) }">
          <span class="initials">{{ getInitials(node.name) }}</span>
        </div>
  
        <div class="card-header">
          <h3 class="name">{{ node.name }}</h3>
          <p class="title">{{ node.title }}</p>
        </div>
  
        <div class="info-section">
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">{{ node.department }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">{{ node.location || 'Toronto, Canada' }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">Level: {{ node.level }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">Management Cost: {{ formatCost(node.managementCost) }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">IC Cost: {{ formatCost(node.icCost) }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">Total Cost: {{ formatCost(node.totalCost) }}</div>
          <div class="info-bubble" :style="{ borderColor: getDepartmentColor(node.department) }">Management Cost Ratio: {{ node.managementCostRatio }}</div>
        </div>
  
        <div class="descendant-count" @click="toggleExpand">
          <p>{{ node.nonLeafDescendants }}/{{ node.descendants }}
            <span v-if="!expanded">↓</span><span v-if="expanded">↑</span>
          </p>
        </div>
      </div>
  
      <div v-if="expanded" class="group-container">
        <div class="children">
          <HierarchyNode v-for="child in node.children" :key="child.id" :node="child" />
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      node: {
        type: Object,
        required: true,
      },
    },
    data() {
      return {
        expanded: false,
      };
    },
    methods: {
      toggleExpand() {
        this.expanded = !this.expanded;
  
        if (this.expanded) {
          this.$nextTick(() => {
            const element = this.$el.querySelector('.card');
            if (element) {
              element.scrollIntoView({
                behavior: 'smooth',
                block: 'center',
                inline: 'center',
              });
            }
          });
        }
      },
      formatCost(cost) {
        return new Intl.NumberFormat('en-CA', {
          style: 'currency',
          currency: 'CAD',
          maximumFractionDigits: 1,
        }).format(cost / 1000);
      },
      getInitials(name) {
        const initials = name.split(' ').map(word => word[0]).join('');
        return initials;
      },
      getDepartmentColor(department) {
        const departmentColors = {
          "Business Intelligence": "#F39C12",
          "Cloud Computing": "#3498DB",
          "Customer Support and Success": "#F90B5F",
          "Cybersecurity": "#2ECC71",
          "Data Analytics": "#0ADDB9",
          "Data Science": "#E67E22",
          "Digital Marketing": "#1ABC9C",
          "Finance and Accounting": "#8E44AD",
          "Human Resources": "#16A085",
          "Information Technology": "#2980B9",
          "Network Operations": "#D35400",
          "Operations and Logistics": "#C0392B",
          "Project Management": "#C40DE2",
          "Quality Assurance": "#27AE60",
          "Sales and Business Development": "#F1C40F",
          "Software Development": "#34495E",
          "Systems Administration": "#7F8C8D",
          "User Experience (UX) Design": "#E74C3C",
          "User Interface (UI) Design": "#2C3E50",
          "Product Management": "#FFDF9A",
        };
        return departmentColors[department] || "#0ADDB9";
      },
    },
  };
  </script>
  
  <style scoped>
  .node {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 0.5rem auto;
  }
  
  .card {
    background-color: rgba(211, 228, 248, 0.15);
    border-radius: 1rem;
    padding: 1.5rem 0.8rem;
    border: 2px solid #0ADDB9;
    box-shadow: 0px 4px 10px 5px rgba(251, 252, 254, 0.20);
    color: #fff;
    width: 240px;
    height: 320px;
    flex-shrink: 0;
    font-family: 'Inter', sans-serif;
    position: relative;
    z-index: 1;
  }
  
  .avatar-circle {
    width: 42px;
    height: 42px;
    background-color: #212830;
    border-radius: 50%;
    border: 2px solid #0ADDB9;
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    top: -21px;
    left: 50%;
    transform: translateX(-50%);
  }
  
  .initials {
    color: #FBFCFE;
    text-align: center;
    font-size: 16px;
    font-weight: 500;
    line-height: 20px;
  }
  
  .card-header {
    margin-top: 6px;
    margin-bottom: 1rem;
  }
  
  .name {
    font-size: 12px;
    font-weight: 500;
    color: #FBFCFE;
    text-align: center;
    line-height: 14px;
  }
  
  .title {
    font-size: 12px;
    font-weight: 500;
    color: #9898AA;
    text-align: center;
    line-height: 35px;
    margin-top: -5px;
  }
  
  .info-section {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .info-bubble {
    display: flex;
    padding: 3px 5px;
    justify-content: center;
    align-items: center;
    align-self: stretch;
    border-radius: 100px;
    border: 1px solid #0ADDB9;
    color: #FBFCFE;
    font-family: 'Inter', sans-serif;
    font-size: 9px;
    font-weight: 500;
    line-height: 12px;
    margin-bottom: 8px;
  }
  
  .descendant-count {
    width: 60px;
    height: 18px;
    flex-shrink: 0;
    border-radius: 100px;
    border: 1px solid #FBFCFE;
    color: #FFF;
    font-family: 'Inter', sans-serif;
    font-size: 10px;
    font-weight: 500;
    line-height: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0.2rem auto 0;
    cursor: pointer;
  }
  

  .group-container {
    border: 2px dashed rgba(255, 255, 255, 0.2);
    padding: 10px;
    border-radius: 1rem;
    margin-top: 2rem;
  }
  
  .children {
    display: flex;
    justify-content: center;
    gap: 2rem;
    flex-direction: row;
  }
  </style>
  