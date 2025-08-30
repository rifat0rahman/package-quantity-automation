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
            <div class="flex items-center justify-between mb-4 flex-wrap gap-4">
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 bg-purple-100 rounded-lg flex items-center justify-center">
                  <i class="fas fa-table text-purple-600"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">Data Preview</h3>
                <div class="badge badge-primary text-white">{{ csvData.length }} rows</div>
              </div>
              <div class="flex items-center gap-3">
                <button @click="startProcessing" :disabled="isProcessing || processedCount > 0"
                  class="btn btn-primary btn-md gap-2 px-6 shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105 text-white">
                  <i class="fas fa-rocket"></i>
                  Start AI Processing
                </button>
                <button v-if="isProcessing" @click="cancelProcessingRun" class="btn btn-error btn-md text-white gap-2">
                  <i class="fas fa-stop-circle"></i> Cancel
                </button>
                <button v-if="!isProcessing && processedCount > 0 && processedCount < rowsToProcess.length"
                  @click="resumeProcessing" class="btn btn-warning btn-md text-white gap-2">
                  <i class="fas fa-play"></i> Resume
                </button>
                <button v-if="!isProcessing && processedCount === rowsToProcess.length && rowsToProcess.length"
                  @click="restartProcessing" class="btn btn-outline btn-md gap-2">
                  <i class="fas fa-undo"></i> Re-run
                </button>
              </div>
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
                      <!-- Only Package Quantity is editable -->
                      <div v-if="col === 'Package Quantity'" class="w-full flex items-center justify-center">
                        <!-- Display Mode -->
                        <div v-if="!isEditingCell(row, col)" @click.stop="enterEdit(row, col)" class="px-2 py-1 rounded cursor-pointer select-none transition
           hover:bg-blue-100 hover:text-blue-700 font-medium tracking-wide
           min-w-[3rem] text-center" :title="'Click to edit ' + col">
                          {{ formatPackageQuantity(row[col]) }}
                        </div>

                        <!-- Edit Mode -->
                        <div v-else class="flex items-center gap-1">
                          <input ref="activeEditor" v-model="editValue" @keydown.enter.prevent="commitEdit()"
                            @keydown.esc.prevent="cancelEdit" @keydown.arrow-up.prevent="stepEdit(1)"
                            @keydown.arrow-down.prevent="stepEdit(-1)" @blur="commitEdit" type="text"
                            inputmode="numeric"
                            class="input input-xs input-bordered w-20 text-center font-semibold tracking-wide"
                            :placeholder="row[col] ? row[col] : '1'" />
                          <div class="flex flex-col -space-y-0.5">
                            <button type="button" class="btn btn-[6px] btn-ghost p-0 h-3 leading-none"
                              style="min-height:0;height:14px;" @mousedown.prevent="stepEdit(1)" :title="'Increase'">
                              <i class="fas fa-chevron-up text-[10px]"></i>
                            </button>
                            <button type="button" class="btn btn-[6px] btn-ghost p-0 h-3 leading-none"
                              style="min-height:0;height:14px;" @mousedown.prevent="stepEdit(-1)" :title="'Decrease'">
                              <i class="fas fa-chevron-down text-[10px]"></i>
                            </button>
                          </div>
                          <button type="button" class="btn btn-ghost btn-xs text-green-600"
                            @mousedown.prevent="commitEdit" :title="'Save'">
                            <i class="fas fa-check"></i>
                          </button>
                          <button type="button" class="btn btn-ghost btn-xs text-red-500"
                            @mousedown.prevent="cancelEdit" :title="'Cancel'">
                            <i class="fas fa-times"></i>
                          </button>
                        </div>
                      </div>

                      <!-- Non-editable cells -->
                      <span v-else-if="isLink(row[col])">
                        <a :href="row[col]" target="_blank" class="text-blue-600 underline">
                          {{ row[col] }}
                        </a>
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
                See More
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
          <div v-if="rowsToProcess.length && (isProcessing || processedCount > 0)" class="my-8">
            <div class="text-center mb-4">
              <div class="flex items-center justify-center gap-3 mb-4">
                <div class="w-8 h-8 bg-orange-100 rounded-lg flex items-center justify-center">
                  <i :class="['fas', isProcessing ? 'fa-cog fa-spin' : 'fa-check-circle', 'text-orange-600']"></i>
                </div>
                <h3 class="text-xl font-bold text-gray-800">
                  {{ isProcessing ? 'Processing Your Data' : 'Processing Complete' }}
                </h3>
              </div>
              <p class="text-gray-600" v-if="isProcessing">
                AI is analyzing product titles and extracting package quantities...
              </p>
              <p v-else class="text-gray-600">
                Finished: {{ processedCount }} rows processed.
              </p>
            </div>
            <div class="max-w-md mx-auto">
              <progress class="progress progress-primary w-full h-4 mb-3" :value="processedCount"
                :max="rowsToProcess.length"></progress>
              <div class="flex justify-between text-sm">
                <span class="text-gray-500">Progress</span>
                <span class="text-primary font-semibold">
                  {{ processedCount }} / {{ rowsToProcess.length }} ({{ percentDone }}%)
                </span>
              </div>
              <div class="text-xs text-gray-500 mt-1">
                Regex classification count: {{ regexProcessedCount }}
              </div>
            </div>
            <div class="flex justify-center mt-6" v-if="isProcessing">
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
      cancelProcessing: false,
      processedCount: 0,
      regexProcessedCount: 0,
      error: "",
      batchSize: 50, // increased default for efficiency
      filterTimeout: null,
      filterOperators: {},
      filterValues: {},
      filteredData: [],
      sortKey: "",
      sortOrder: "",
      visibleRows: 50,
      defaultVisibleRows: 50,
      config: {
        treatBareNumberPackAsMultipack: false,
        maxReasonablePackageQty: 200
      },
      // Inline editing state
      editing: {
        row: null,
        col: null
      },
      editValue: ""
    };
  },
  computed: {
    paginatedFilteredData() {
      return this.filteredData.slice(0, this.visibleRows);
    },
    percentDone() {
      if (!this.rowsToProcess.length) return 0;
      return ((this.processedCount / this.rowsToProcess.length) * 100).toFixed(1);
    }
  },
  methods: {
    async onFileChange(e) {
      this.error = "";
      const file = e.target.files[0];
      if (!file) return;

      // Reset state
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
      this.cancelEdit();
      this.cancelProcessing = false;
      this.isProcessing = false;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          this.csvData = results.data;
          this.columns = results.meta.fields;

          if (!this.columns.includes("Package Quantity")) {
            this.columns.push("Package Quantity");
            this.csvData.forEach(r => { if (r["Package Quantity"] === undefined) r["Package Quantity"] = ""; });
          }
          if (!this.columns.includes("Unit Count")) {
            this.columns.push("Unit Count");
            this.csvData.forEach(r => { if (r["Unit Count"] === undefined) r["Unit Count"] = ""; });
          }

          this.rowsToProcess = this.csvData.filter((row) => row["Title"]);

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

    classifyTitle(title) {
      if (!title || typeof title !== 'string') {
        return { packageQuantity: 1, unitCount: null };
      }
      const lower = title.toLowerCase();

      const multipackWordNumbers = {
        twin: 2, double: 2, duo: 2, triple: 3, triplet: 3, quad: 4, quadruple: 4
      };

      const unitContentWords = [
        "count", "ct", "tablet", "tablets", "capsule", "capsules", "caplet", "caplets", "softgel", "softgels",
        "gummy", "gummies", "drop", "drops", "wipe", "wipes", "pad", "pads", "liner", "liners", "tampon", "tampons",
        "crayon", "crayons", "pencil", "pencils", "marker", "markers", "tip", "tips", "filter", "filters", "roll", "rolls",
        "sponge", "sponges", "sheet", "sheets", "strip", "strips", "stick", "sticks", "bar", "bars", "sachet", "sachets",
        "packet", "packets", "pouch", "pouches", "serving", "servings", "teaspoon", "teaspoons", "underpad", "underpads",
        "cartridge", "cartridges", "lancet", "lancets", "batteries", "battery", "pairs", "pair"
      ];

      const partNumberMarkers = [
        "mfrpartno", "partno", "part#", "model", "model#", "sku", "asin", "upc", "mpn", "mfr", "manufacturer", "item#"
      ];

      const multiPackIndicators = [];
      const unitCountCandidates = [];

      const MAX_QTY = this.config?.maxReasonablePackageQty ?? 200;
      const HIGH_SANITY_LIMIT = 50; // tighter sanity threshold

      // 1. Explicit "pack of N"
      const packOfRegex = /\bpack\s+of\s+(\d+)\b/i;
      const packOfMatch = title.match(packOfRegex);
      if (packOfMatch) {
        const n = +packOfMatch[1];
        if (n > 0 && n <= MAX_QTY) multiPackIndicators.push({ value: n, type: "pack_of" });
      }

      // 2. N-pack variants
      const nPackRegex = /\b(\d+)\s*-?\s*pack(s)?\b/gi;
      let nMatch;
      while ((nMatch = nPackRegex.exec(title)) !== null) {
        const n = +nMatch[1];
        if (n > 0 && n <= MAX_QTY) multiPackIndicators.push({ value: n, type: "n_pack" });
      }

      // 3. Word-based
      const wordPackRegex = new RegExp("\\b(" + Object.keys(multipackWordNumbers).join("|") + ")\\s+pack\\b", "i");
      const wordPackMatch = title.match(wordPackRegex);
      if (wordPackMatch) {
        const val = multipackWordNumbers[wordPackMatch[1].toLowerCase()];
        if (val > 0 && val <= MAX_QTY) multiPackIndicators.push({ value: val, type: "word_pack" });
      }

      // 4. Exclusions: dimensions/resolutions & part numbers
      const resolutionContextWords = /(display|screen|monitor|resolution|touchscreen|laptop|notebook|camera|pixel|px)\b/;
      const resolutionLike = /\b(\d{3,5})\s*[x×]\s*(\d{3,5})\b/i;
      const isResolution = resolutionLike.test(lower) && resolutionContextWords.test(lower);

      // detect presence of part number marker near a big x-pattern token
      const hasPartNumberMarker = partNumberMarkers.some(m => lower.includes(m));

      // 5. Safer x-pattern (integer x integer), only if NOT a resolution or partnumber context
      // Allow a small outer dimension (<= MAX_QTY) OR explicit suffix
      const packXPattern = /\b(\d+)\s*[x×]\s*(\d+)(?:\s*(count|ct|pack|packs|pk|pks))?\b/gi;
      if (!isResolution) {
        let xp;
        while ((xp = packXPattern.exec(lower)) !== null) {
          const outer = +xp[1];
          const inner = +xp[2];
          const suffix = xp[3];

          // Reject if looks like model/part number pair (very large) or partnumber marker present
          if (outer >= 1000 || inner >= 1000) continue;
          if (hasPartNumberMarker && (outer > 50 || inner > 50) && !suffix) continue;

          // Reject if outer has 4+ digits (already covered above) or ratio suggests a resolution (outer>=200 && inner>=200)
          if ((outer >= 200 && inner >= 200) && !suffix) continue;

          if (outer > 0 && outer <= MAX_QTY) {
            multiPackIndicators.push({ value: outer, type: "x_pattern" });
            if (suffix) {
              // treat inner as per-unit count only when suffix shows it's a pack-like context
              if (inner > 0) unitCountCandidates.push({ value: inner });
            }
          }
        }
      }

      // 6. Unit count words
      const unitRegex = new RegExp("\\b(\\d{1,5})\\s*(" + unitContentWords.join("|") + ")\\b", "gi");
      let uc;
      while ((uc = unitRegex.exec(lower)) !== null) {
        const val = +uc[1];
        if (val > 0) unitCountCandidates.push({ value: val });
      }

      // Determine packageQuantity vs unitCount
      let packageQuantity = 1;
      let unitCount = null;

      if (multiPackIndicators.length) {
        const priority = { pack_of: 1, x_pattern: 2, n_pack: 3, word_pack: 4 };
        multiPackIndicators.sort((a, b) => priority[a.type] - priority[b.type]);

        const chosen = multiPackIndicators[0];

        if (chosen.type === "n_pack" &&
          !this.config?.treatBareNumberPackAsMultipack &&
          unitCountCandidates.length === 0 &&
          multiPackIndicators.length === 1) {
          unitCount = chosen.value;
          packageQuantity = 1;
        } else {
          packageQuantity = chosen.value;
        }
      }

      if (unitCount == null && unitCountCandidates.length) {
        unitCountCandidates.sort((a, b) => b.value - a.value);
        unitCount = unitCountCandidates[0].value;
      }

      // Sanity demotions
      if (packageQuantity > HIGH_SANITY_LIMIT &&
        !['pack_of', 'n_pack', 'word_pack'].includes(multiPackIndicators[0]?.type)) {
        packageQuantity = 1;
      }
      if (packageQuantity > MAX_QTY) {
        packageQuantity = 1;
      }

      return { packageQuantity, unitCount };
    },

    startProcessing() {
      if (this.isProcessing) return;
      if (!this.rowsToProcess.length) {
        this.rowsToProcess = this.csvData.filter(r => r["Title"]);
        if (!this.rowsToProcess.length) return;
      }
      this.error = "";
      this.cancelProcessing = false;
      this.isProcessing = true;
      // If re-run, reset counters
      if (this.processedCount === this.rowsToProcess.length) {
        this.processedCount = 0;
        this.regexProcessedCount = 0;
        // Clear previously computed fields if you want a clean rerun:
        // this.rowsToProcess.forEach(r => { r["Package Quantity"] = ""; r["Unit Count"] = ""; });
      }
      // Begin async batch loop
      this.processNextBatch(this.processedCount); // resume from where left off
    },

    resumeProcessing() {
      if (this.isProcessing) return;
      if (this.processedCount >= this.rowsToProcess.length) return;
      this.cancelProcessing = false;
      this.isProcessing = true;
      this.processNextBatch(this.processedCount);
    },

    restartProcessing() {
      this.processedCount = 0;
      this.regexProcessedCount = 0;
      this.rowsToProcess = this.csvData.filter(r => r["Title"]);
      this.rowsToProcess.forEach(r => {
        // Optionally wipe previous results
        r["Package Quantity"] = "";
        r["Unit Count"] = "";
      });
      this.startProcessing();
    },

    cancelProcessingRun() {
      this.cancelProcessing = true;
    },

    processNextBatch(startIndex) {
      if (this.cancelProcessing) {
        this.isProcessing = false;
        return;
      }
      if (startIndex >= this.rowsToProcess.length) {
        // Done
        this.isProcessing = false;
        this.filteredData = [...this.csvData];
        return;
      }

      const end = Math.min(startIndex + this.batchSize, this.rowsToProcess.length);
      const batch = this.rowsToProcess.slice(startIndex, end);

      batch.forEach(row => {
        try {
          const title = row["Title"];
          const { packageQuantity, unitCount } = this.classifyTitle(title);
          row["Package Quantity"] = packageQuantity;
          if (unitCount !== null) row["Unit Count"] = unitCount;
          this.regexProcessedCount++;
        } catch {
          row["Package Quantity"] = 1;
        }
        this.processedCount++;
      });

      // Ensure reactive update & yield
      this.$nextTick(() => {
        setTimeout(() => {
          this.processNextBatch(end);
        }, 0);
      });
    },

    parseNumeric(val) {
      if (val === null || val === undefined) return NaN;
      if (typeof val === "number") return val;
      let str = String(val).trim();
      str = str
        .replace(/[\$,]/g, '')
        .replace(/^\((.*)\)$/, '-$1')
        .replace(/[%]/g, '')
        .replace(/[^0-9.\-]/g, ' ')
        .trim()
        .split(/\s+/)[0];
      return parseFloat(str);
    },

    isNumericColumn(col) {
      if (!this.csvData.length) return false;
      if (col.toLowerCase().includes("quantity") || col.toLowerCase().includes("count") ||
        col.toLowerCase().includes("price") || col.toLowerCase().includes("cost") ||
        col.toLowerCase().includes("unit")) return true;
      const sample = this.csvData.slice(0, 10).map(r => this.parseNumeric(r[col]));
      const numericSamples = sample.filter(v => !isNaN(v));
      return numericSamples.length >= Math.ceil(sample.length * 0.5);
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
            const actualNum = this.parseNumeric(actual);
            const value = this.parseNumeric(filterVal);
            if (isNaN(value) || isNaN(actualNum)) return false;
            switch (operator) {
              case '>': return actualNum > value;
              case '<': return actualNum < value;
              case '=': return actualNum === value;
              case '>=': return actualNum >= value;
              case '<=': return actualNum <= value;
              default: return true;
            }
          } else {
            return String(actual ?? '').toLowerCase().includes(String(filterVal).toLowerCase());
          }
        });
      });
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
      this.filteredData = [...this.filteredData].sort((a, b) => {
        let valA = a[key], valB = b[key];
        if (this.isNumericColumn(key)) {
          valA = this.parseNumeric(valA); valB = this.parseNumeric(valB);
          if (isNaN(valA)) valA = -Infinity;
          if (isNaN(valB)) valB = -Infinity;
          return (valA - valB) * modifier;
        } else {
          return String(valA ?? '').localeCompare(String(valB ?? '')) * modifier;
        }
      });
    },

    showMoreRows() {
      this.visibleRows += 50;
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
      this.commitEditIfOpen();
      const csv = Papa.unparse(this.filteredData);
      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.setAttribute("download", "filtered_products.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    /* ---------- Editable Package Quantity Cell Logic ---------- */
    isEditingCell(row, col) {
      return this.editing.row === row && this.editing.col === col;
    },

    enterEdit(row, col) {
      this.commitEditIfOpen();
      this.editing.row = row;
      this.editing.col = col;
      this.editValue = (row[col] === undefined || row[col] === null || row[col] === "") ? "" : String(row[col]);
      this.$nextTick(() => {
        if (this.$refs.activeEditor) {
          const el = Array.isArray(this.$refs.activeEditor) ? this.$refs.activeEditor[0] : this.$refs.activeEditor;
          if (el && el.focus) {
            el.focus();
            el.select && el.select();
          }
        }
      });
    },

    sanitizeQuantity(val) {
      if (val === null || val === undefined) return 1;
      let s = String(val).trim();
      if (s === "") return 1;
      s = s.replace(/[^\d]/g, '');
      if (s === "") return 1;
      let n = parseInt(s, 10);
      if (isNaN(n) || n <= 0) n = 1;
      if (n > this.config.maxReasonablePackageQty) n = this.config.maxReasonablePackageQty;
      return n;
    },

    commitEdit() {
      if (!this.editing.row || !this.editing.col) return;
      const row = this.editing.row;
      const col = this.editing.col;
      const qty = this.sanitizeQuantity(this.editValue);
      row[col] = qty;
      this.cancelEdit();
      this.applyFilters();
    },

    commitEditIfOpen() {
      if (this.editing.row && this.editing.col) {
        this.commitEdit();
      }
    },

    cancelEdit() {
      this.editing.row = null;
      this.editing.col = null;
      this.editValue = "";
    },

    stepEdit(delta) {
      let current = this.sanitizeQuantity(this.editValue === "" ? 1 : this.editValue);
      current += delta;
      if (current < 1) current = 1;
      if (current > this.config.maxReasonablePackageQty) current = this.config.maxReasonablePackageQty;
      this.editValue = String(current);
    },

    formatPackageQuantity(val) {
      if (val === undefined || val === null || val === "") return "-";
      return val;
    },

    onOutsideClick(e) {
      if (!this.editing.row) return;
      const editingEls = this.$refs.activeEditor;
      if (editingEls) {
        const el = Array.isArray(editingEls) ? editingEls[0] : editingEls;
        if (el && !el.contains(e.target)) {
          this.commitEdit();
        }
      }
    },
  },
  mounted() {
    document.addEventListener("click", this.onOutsideClick);
  },
  beforeUnmount() {
    document.removeEventListener("click", this.onOutsideClick);
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

.input:focus {
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.35);
}

/* Optional: if your Tailwind setup doesn't already define animate-blob */
@keyframes blob {

  0%,
  100% {
    transform: translate(0px, 0px) scale(1);
  }

  33% {
    transform: translate(30px, -20px) scale(1.05);
  }

  66% {
    transform: translate(-20px, 20px) scale(0.95);
  }
}

.animate-blob {
  animation: blob 12s infinite;
}

.animation-delay-2000 {
  animation-delay: 2s;
}

.animation-delay-4000 {
  animation-delay: 4s;
}
</style>