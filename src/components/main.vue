<template>
  <div data-theme="light"
    class="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-50 relative overflow-hidden">
    <!-- Animated background elements -->
    <div class="absolute inset-0 opacity-30">
      <div
        class="absolute top-20 left-20 w-72 h-72 bg-blue-200 rounded-full mix-blend-multiply filter blur-xl animate-blob">
      </div>
      <div
        class="absolute top-40 right-20 w-72 h-72 bg-purple-200 rounded-full mix-blend-multiply filter blur-xl animate-blob animation-delay-2000">
      </div>
      <div
        class="absolute bottom-20 left-40 w-72 h-72 bg-pink-200 rounded-full mix-blend-multiply filter blur-xl animate-blob animation-delay-4000">
      </div>
    </div>

    <div class="relative z-10 flex flex-col items-center py-10 px-4">
      <!-- Hero Header -->
      <div class="text-center mb-12">
        <div class="inline-flex items-center gap-3 mb-4">
          <div
            class="w-12 h-12 bg-gradient-to-r from-blue-500 to-purple-600 rounded-xl flex items-center justify-center shadow-lg">
            <i class="fas fa-robot text-white text-xl"></i>
          </div>
          <h1
            class="text-5xl md:text-6xl font-black bg-gradient-to-r from-blue-600 via-purple-600 to-pink-600 bg-clip-text text-transparent">
            Amazon Wholesale</h1>
        </div>
        <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-4">CSV Claude Automation</h2>
        <p class="text-lg text-gray-600 max-w-2xl mx-auto leading-relaxed">
          Transform your product data with AI-powered package quantity detection. Professional-grade automation for
          wholesale businesses.
        </p>
        <!-- Stats -->
        <div class="flex flex-wrap justify-center gap-8 mt-8">
          <div class="text-center">
            <div class="text-3xl font-bold text-blue-600">99.9%</div>
            <div class="text-sm text-gray-500">Accuracy</div>
          </div>
          <div class="text-center">
            <div class="text-3xl font-bold text-purple-600">10x</div>
            <div class="text-sm text-gray-500">Faster</div>
          </div>
          <div class="text-center">
            <div class="text-3xl font-bold text-pink-600">AI</div>
            <div class="text-sm text-gray-500">Powered</div>
          </div>
        </div>
      </div>

      <!-- Main Processing Card -->
      <div class="card w-full  bg-white/80 backdrop-blur-xl shadow-2xl border border-gray-200/50">
        <div class="card-body p-8">

          <!-- Upload Section -->
          <div class="grid md:grid-cols-2 gap-8 mb-8">
            <div class="space-y-4">
              <div class="flex items-center gap-3 mb-4">
                <div class="w-8 h-8 bg-blue-100 rounded-lg flex items-center justify-center">
                  <i class="fas fa-upload text-blue-600"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">Upload Your CSV</h3>
              </div>

              <div class="form-control">
                <label class="label">
                  <span class="label-text text-gray-700 font-medium">Select CSV File</span>
                </label>
                <div class="relative">
                  <input type="file" accept=".csv" @change="onFileChange"
                    class="file-input file-input-bordered file-input-primary w-full bg-gray-50 border-gray-300 text-gray-700 hover:bg-gray-100 transition-all duration-300" />
                  <div class="absolute right-4 top-1/2 -translate-y-1/2 pointer-events-none">
                    <i class="fas fa-file-csv text-primary"></i>
                  </div>
                </div>
              </div>
            </div>

            <!-- Features -->
            <div class="space-y-3">
              <h4 class="font-semibold text-gray-800 mb-4">✨ What you get:</h4>
              <div class="space-y-3">
                <div class="flex items-center gap-3 text-gray-600">
                  <i class="fas fa-check-circle text-green-500"></i>
                  <span>AI-powered quantity detection</span>
                </div>
                <div class="flex items-center gap-3 text-gray-600">
                  <i class="fas fa-check-circle text-green-500"></i>
                  <span>Batch processing optimization</span>
                </div>
                <div class="flex items-center gap-3 text-gray-600">
                  <i class="fas fa-check-circle text-green-500"></i>
                  <span>Error handling & fallbacks</span>
                </div>
                <div class="flex items-center gap-3 text-gray-600">
                  <i class="fas fa-check-circle text-green-500"></i>
                  <span>Instant CSV download</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Preview Section -->
          <div v-if="csvData.length" class="mb-8">
            <div class="flex items-center justify-between mb-4">
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 bg-purple-100 rounded-lg flex items-center justify-center">
                  <i class="fas fa-table text-purple-600"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">Data Preview</h3>
                <div class="badge badge-primary text-white">{{ csvData.length }} rows</div>
              </div>
              <button @click="startProcessing" :disabled="isProcessing || processedCount > 0"
                class="btn btn-primary btn-md gap-2 px-6 shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105 text-white ml-4">
                <i class="fas fa-rocket"></i>
                Start AI Processing
              </button>
            </div>

            <div class="overflow-x-auto rounded-xl bg-white shadow-sm" style="position:relative;">
              <table class="table table-sm table-pin-rows">
                <thead class="bg-gray-50">
                  <tr>
                    <th v-for="col in columns" :key="col" class="text-gray-700 font-semibold">
                      <div class="flex flex-col items-center space-y-1 w-full">
                        <span @click="sortByColumn(col)" class="cursor-pointer flex items-center gap-1">
                          {{ col }}
                          <span v-if="sortKey === col" class="ml-1">
                            {{ sortOrder === 'asc' ? '▲' : '▼' }}
                          </span>
                        </span>
                        <div v-if="isNumericColumn(col)" class="flex items-center filter-group">
                          <select v-model="filterOperators[col]"
                            class="filter-operator bg-gray-100 border border-gray-300 rounded-l text-xs px-2 py-1">
                            <option value=">">&gt;</option>
                            <option value="<">&lt;</option>
                            <option value="=">=</option>
                            <option value=">=">&gt;=</option>
                            <option value="<=">&lt;=</option>
                          </select>
                          <input v-model="filterValues[col]" @input="debounceApplyFilters"
                            class="filter-value border border-l-0 border-gray-300 rounded-r text-xs px-2 py-1 w-16 focus:outline-none"
                            placeholder="Value">
                        </div>
                        <div v-else class="flex items-center filter-group">
                          <input v-model="filterValues[col]" @input="debounceApplyFilters"
                            class="filter-value border border-gray-300 rounded text-xs px-2 py-1 w-20 focus:outline-none"
                            placeholder="Filter">
                        </div>
                      </div>
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, idx) in paginatedFilteredData" :key="idx" class="hover:bg-blue-50 transition-colors">
                    <td v-for="col in columns" :key="col" class="text-gray-600">
                      <span v-if="isLink(row[col])">
                        <a :href="row[col]" target="_blank" class="text-blue-600 underline">{{ row[col] }}</a>
                      </span>
                      <span v-else>{{ row[col] }}</span>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>

            <div class="mt-4 flex justify-center gap-2">
              <button class="btn btn-warning text-white btn-sm" @click="showMoreRows"
                v-if="visibleRows < filteredData.length">
                See More ({{ filteredData.length - visibleRows }} left)
              </button>
              <button class="btn btn-info text-white btn-sm" @click="resetRows" v-if="visibleRows > defaultVisibleRows">
                Reset Rows
              </button>
            </div>

            <div class="mt-6 flex justify-center gap-4">
              <button @click="downloadFilteredCSV"
                class="btn btn-success btn-lg gap-3 px-8 shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105 text-white">
                <i class="fas fa-download"></i>
                Download Filtered CSV
              </button>
            </div>
          </div>

          <!-- Progress Section -->
          <div v-if="isProcessing" class="my-8">
            <div class="text-center mb-4">
              <div class="flex items-center justify-center gap-3 mb-4">
                <div class="w-8 h-8 bg-orange-100 rounded-lg flex items-center justify-center">
                  <i class="fas fa-cog fa-spin text-orange-600"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">Processing Your Data</h3>
              </div>
              <p class="text-gray-600">AI is analyzing product titles and extracting package quantities...</p>
            </div>
            <div class="max-w-md mx-auto">
              <progress class="progress progress-primary w-full h-4 mb-3" :value="processedCount"
                :max="rowsToProcess.length"></progress>
              <div class="flex justify-between text-sm">
                <span class="text-gray-500">Progress</span>
                <span class="text-primary font-semibold">{{ processedCount }} / {{ rowsToProcess.length }} rows</span>
              </div>
            </div>
            <!-- Processing animation -->
            <div class="flex justify-center mt-6">
              <div class="flex gap-2">
                <div class="w-3 h-3 bg-blue-500 rounded-full animate-bounce"></div>
                <div class="w-3 h-3 bg-purple-500 rounded-full animate-bounce" style="animation-delay: 0.1s"></div>
                <div class="w-3 h-3 bg-pink-500 rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
              </div>
            </div>
          </div>

          <!-- Error Section -->
          <div v-if="error" class="alert alert-error shadow-lg mt-4">
            <div class="flex items-center gap-3">
              <i class="fas fa-exclamation-triangle text-xl"></i>
              <div>
                <h4 class="font-semibold">Processing Error</h4>
                <p class="text-sm opacity-80">{{ error }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import Papa from "papaparse";

export default {
  name: "App",
  data() {
    return {
      csvData: [],
      columns: [],
      rowsToProcess: [],
      isProcessing: false,
      processedCount: 0,
      regexProcessedCount: 0,
      error: "",
      batchSize: 5,
      filterTimeout: null,
      filterOperators: {},
      filterValues: {},
      filteredData: [],
      sortKey: "",
      sortOrder: "",
      visibleRows: 50,
      defaultVisibleRows:5,
    };
  },
  computed: {
    paginatedFilteredData() {
      return this.filteredData.slice(0, this.visibleRows);
    }
  },
  methods: {
    async onFileChange(e) {
      this.error = "";
      const file = e.target.files[0];
      if (!file) return;

      // Clear previous data before parsing new file
      this.csvData = [];
      this.columns = [];
      this.rowsToProcess = [];
      this.processedCount = 0;
      this.regexProcessedCount = 0;
      this.filterOperators = {};
      this.filterValues = {};
      this.filteredData = [];
      this.sortKey = "";
      this.sortOrder = "";
      this.visibleRows = this.defaultVisibleRows;

      // Remove previous localStorage
      localStorage.removeItem("csvData");
      localStorage.removeItem("columns");

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          this.csvData = results.data;
          this.columns = results.meta.fields;
          this.rowsToProcess = this.csvData.filter((row) => row["Title"]);

          // Store in localStorage
          localStorage.setItem("csvData", JSON.stringify(this.csvData));
          localStorage.setItem("columns", JSON.stringify(this.columns));

          // Setup filter defaults
          this.columns.forEach(col => {
            this.filterOperators[col] = this.isNumericColumn(col) ? '>' : '';
            this.filterValues[col] = '';
          });

          this.filteredData = [...this.csvData];
        },
        error: (err) => {
          this.error = "CSV Parse error: " + err.message;
        },
      });
    },

    extractQuantityFromTitle(title) {
      if (!title || typeof title !== 'string') {
        return null;
      }
      // High confidence patterns (priority order)
      const highConfidencePatterns = [
        /Pack of (\d+)/i,
        /\(Pack of (\d+)\)/i,
        /(\d+)-Pack/i,
        /(\d+) Pack(?!\s*(?:oz|ml|mg|g|lb|fl|ounce|inches?))/i,
        /(\d+) Count/i,
        /(\d+)-Count/i,
        /\((\d+) Pack\)/i,
      ];
      // Medium confidence patterns
      const mediumConfidencePatterns = [
        /(\d+) ea\b/i,
        /(\d+) ct\b/i,
        /(\d+) Bars?\b/i,
        /(\d+) Bottles?\b/i,
        /(\d+) Drops?\b/i,
        /(\d+) Tablets?\b/i,
        /(\d+) Capsules?\b/i,
        /(\d+) Count\b/i,
        /(\d+)-ea\b/i,
        /(\d+) Each\b/i,
        /(\d+) Units?\b/i,
        /(\d+) Pieces?\b/i,
        /Set of (\d+)/i,
        /(\d+) Kit/i,
      ];
      // Word number mappings
      const wordNumbers = {
        'twin pack': 2,
        'double pack': 2,
        'triple pack': 3,
        'dozen': 12,
        'half dozen': 6,
        'pair': 2,
        'set of two': 2,
        'set of three': 3,
        'set of four': 4,
        'set of five': 5,
        'set of six': 6,
      };
      for (const pattern of highConfidencePatterns) {
        const match = title.match(pattern);
        if (match) {
          const qty = parseInt(match[1], 10);
          if (!isNaN(qty) && qty > 0 && qty <= 1000) {
            return qty;
          }
        }
      }
      for (const pattern of mediumConfidencePatterns) {
        const match = title.match(pattern);
        if (match) {
          const qty = parseInt(match[1], 10);
          if (!isNaN(qty) && qty > 0 && qty <= 1000) {
            return qty;
          }
        }
      }
      const lowerTitle = title.toLowerCase();
      for (const [phrase, qty] of Object.entries(wordNumbers)) {
        if (lowerTitle.includes(phrase)) {
          return qty;
        }
      }
      return 1;
    },

    async startProcessing() {
      this.error = "";
      this.isProcessing = true;
      this.processedCount = 0;
      this.regexProcessedCount = 0;

      try {
        for (let i = 0; i < this.rowsToProcess.length; i += this.batchSize) {
          const batch = this.rowsToProcess.slice(i, i + this.batchSize);

          batch.forEach((row) => {
            try {
              const regexQuantity = this.extractQuantityFromTitle(row["Title"]);
              row["Package Quantity"] = regexQuantity;
              this.regexProcessedCount++;
              this.processedCount++;
              // Add column if not present
              if (!this.columns.includes("Package Quantity")) {
                this.columns.push("Package Quantity");
                this.filterOperators["Package Quantity"] = '>';
                this.filterValues["Package Quantity"] = '';
              }
            } catch (err) {
              row["Package Quantity"] = 1; // fallback
              this.processedCount++;
            }
          });

          // Small delay between batches
          if (i + this.batchSize < this.rowsToProcess.length) {
            await new Promise(resolve => setTimeout(resolve, 200));
          }
        }
        // Update filtered data after processing
        this.filteredData = [...this.csvData];
        localStorage.setItem("csvData", JSON.stringify(this.csvData));
        localStorage.setItem("columns", JSON.stringify(this.columns));
      } catch (err) {
        this.error = "Processing error: " + err.message;
      }
      this.isProcessing = false;
    },

    isNumericColumn(col) {
      // Heuristic: sample first row, allow Package Quantity, numbers
      if (!this.csvData.length) return false;
      const val = this.csvData[0][col];
      if (col.toLowerCase().includes("quantity") || col.toLowerCase().includes("price") || col.toLowerCase().includes("cost") || col.toLowerCase().includes("count") || col.toLowerCase().includes("volume")) {
        return true;
      }
      return typeof val === "number" || (!isNaN(parseFloat(val)) && isFinite(val));
    },

    debounceApplyFilters() {
      clearTimeout(this.filterTimeout);
      this.filterTimeout = setTimeout(() => {
        this.applyFilters();
      }, 250);
    },

    applyFilters() {
      this.filteredData = this.csvData.filter(row => {
        return this.columns.every(key => {
          const filterVal = this.filterValues[key];
          if (!filterVal) return true;
          const operator = this.filterOperators[key];
          let actual = row[key];
          if (this.isNumericColumn(key)) {
            actual = parseFloat(actual);
            const value = parseFloat(filterVal);
            if (isNaN(value)) return true;
            switch (operator) {
              case '>': return actual > value;
              case '<': return actual < value;
              case '=': return actual === value;
              case '>=': return actual >= value;
              case '<=': return actual <= value;
              default: return true;
            }
          } else {
            // Non-numeric: substring match (case-insensitive)
            return String(actual).toLowerCase().includes(String(filterVal).toLowerCase());
          }
        });
      });
      // re-apply sort if exists
      if (this.sortKey) {
        this.sortByColumn(this.sortKey, true);
      }
      this.visibleRows = this.defaultVisibleRows;
    },

    sortByColumn(key, skipToggle = false) {
      if (!skipToggle) {
        if (this.sortKey === key) {
          this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
        } else {
          this.sortKey = key;
          this.sortOrder = 'asc';
        }
      }
      const modifier = this.sortOrder === 'desc' ? -1 : 1;
      // Sort filteredData
      this.filteredData = [...this.filteredData].sort((a, b) => {
        let valA = a[key], valB = b[key];
        if (this.isNumericColumn(key)) {
          valA = parseFloat(valA); valB = parseFloat(valB);
          return (valA - valB) * modifier;
        } else {
          return String(valA).localeCompare(String(valB)) * modifier;
        }
      });
    },

    showMoreRows() {
      this.visibleRows += 50;
    },
    resetRows() {
      this.visibleRows = this.defaultVisibleRows;
    },

    isLink(val) {
      if (!val) return false;
      if (typeof val !== "string") return false;
      return /^https?:\/\/.+\..+/.test(val.trim());
    },

    downloadFilteredCSV() {
      if (!this.filteredData.length) {
        alert('No results to download');
        return;
      }
      const csv = Papa.unparse(this.filteredData);
      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.setAttribute("download", "filtered_products.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    pushToGoogleSheet() {
      // Open Google Sheets with the content as a new sheet via CSV upload
      this.downloadFilteredCSV();
      window.open("https://docs.google.com/spreadsheets/u/0/", "_blank");
    },
  },
  mounted() {
    // Load from localStorage if exists
    const storedCsv = localStorage.getItem("csvData");
    const storedCols = localStorage.getItem("columns");
    if (storedCsv && storedCols) {
      try {
        this.csvData = JSON.parse(storedCsv);
        this.columns = JSON.parse(storedCols);
        this.rowsToProcess = this.csvData.filter((row) => row["Title"]);
        this.filterOperators = {};
        this.filterValues = {};
        this.columns.forEach(col => {
          this.filterOperators[col] = this.isNumericColumn(col) ? '>' : '';
          this.filterValues[col] = '';
        });
        this.filteredData = [...this.csvData];
        this.visibleRows = this.defaultVisibleRows;
      } catch { }
    }
  }
};
</script>

<style scoped>
.filter-group {
  width: 100%;
  justify-content: center;
}

.filter-operator {
  min-width: 32px;
  height: 28px;
  border-radius: 6px 0 0 6px;
}

.filter-value {
  min-width: 40px;
  height: 28px;
  border-radius: 0 6px 6px 0;
}

.table thead th {
  position: sticky;
  top: 0;
  z-index: 30;
  background: #f8f9fa;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.table th {
  padding-bottom: 0.75rem !important;
}
.table-pin-rows thead th {
  position: sticky !important;
  top: 0 !important;
  z-index: 10 !important;
}
</style>
