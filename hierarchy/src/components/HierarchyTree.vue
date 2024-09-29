<template>
  <div class="hierarchy-tree" ref="treeContainer">
    <HierarchyNode v-if="rootNode" :node="rootNode" @expand="scrollToNode" />
  </div>
</template>

<script>
import Papa from 'papaparse';
import HierarchyNode from './HierarchyNode.vue';

export default {
  components: {
    HierarchyNode,
  },
  data() {
    return {
      rootNode: null,
    };
  },
  mounted() {
    this.loadCSV('/data.csv');
  },
  methods: {
    loadCSV(filePath) {
      fetch(filePath)
        .then(response => response.text())
        .then(csvText => {
          Papa.parse(csvText, {
            header: true,
            skipEmptyLines: true,
            complete: (results) => {
              this.processData(results.data);
            },
            error: (error) => {
              console.error('Error parsing CSV: ', error);
            },
          });
        })
        .catch(error => {
          console.error('Error loading CSV file: ', error);
        });
    },
    processData(data) {
      const employees = this.buildHierarchy(data);
      const rootNode = employees.find(employee => !employee.managerId);
      this.calculateCosts(rootNode);
      this.rootNode = rootNode;
    },
    buildHierarchy(data) {
      const employees = data.map(item => ({
        id: item['Employee Id'],
        name: item['Name'],
        title: item['Job Title'],
        department: item['Department'],
        location: item['Location'] || 'Toronto, Canada',
        salary: parseFloat(item['Salary'].replace(/[^0-9.-]+/g,"")),
        level: item['level'],
        managerId: item['Manager'],
        children: [],
        managementCost: 0,
        icCost: 0,
        totalCost: 0,
        managementCostRatio: 0,
        descendants: 0,
        nonLeafDescendants: 0,
      }));

      const employeeMap = {};
      employees.forEach(employee => {
        employeeMap[employee.id] = employee;
      });

      employees.forEach(employee => {
        if (employee.managerId && employeeMap[employee.managerId]) {
          const manager = employeeMap[employee.managerId];
          manager.children.push(employee);
        }
      });

      return employees;
    },
    calculateCosts(node) {
      let totalCost = node.salary;
      let managementCost = 0;
      let icCost = 0;
      let descendants = 0;
      let nonLeafDescendants = 0;

      node.children.forEach(child => {
        const childCosts = this.calculateCosts(child);
        descendants += childCosts.descendants + 1;

        if (childCosts.descendants > 0) {
          managementCost += childCosts.totalCost;
          nonLeafDescendants++;
        } else {
          icCost += childCosts.totalCost;
        }

        totalCost += childCosts.totalCost;
      });

      node.managementCost = managementCost;
      node.icCost = icCost;
      node.totalCost = totalCost;
      node.descendants = descendants;
      node.nonLeafDescendants = nonLeafDescendants;
      node.managementCostRatio = ((managementCost / totalCost)|| 0).toFixed(2);

      return {
        totalCost,
        managementCost,
        icCost,
        descendants,
        nonLeafDescendants,
      };
    },
    scrollToNode(element) {
      const container = this.$refs.treeContainer;

      const containerWidth = container.offsetWidth;
      const containerScrollLeft = container.scrollLeft;
      const containerCenter = containerScrollLeft + containerWidth / 2;

      const elementWidth = element.offsetWidth;
      const elementOffsetLeft = element.offsetLeft;
      const elementCenter = elementOffsetLeft + elementWidth / 2;

      const scrollLeft = containerScrollLeft + (elementCenter - containerCenter);
      container.scrollTo({
        left: scrollLeft,
        behavior: 'smooth',
      });
    },
  },
};
</script>

<style scoped>
.hierarchy-tree {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
  min-height: 100vh;
  padding: 2rem;
  position: relative;
  overflow-x: scroll;
}
</style>
