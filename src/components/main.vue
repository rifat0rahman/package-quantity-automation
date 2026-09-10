<template>
  <div data-theme="light" class="min-h-screen bg-gray-100 text-gray-900">

    <!-- ===== Top Bar ===== -->
    <header class="bg-white border-b border-gray-200 sticky top-0 z-30">
      <div class="mx-auto w-full px-4 py-3 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="w-9 h-9 bg-blue-600 rounded-lg flex items-center justify-center">
            <span class="text-white font-bold text-lg">Q</span>
          </div>
          <div>
            <h1 class="text-base font-semibold leading-tight">Package Quantity Automation</h1>
            <p class="text-xs text-gray-500">Amazon wholesale product data processing</p>
          </div>
        </div>
        <a href="https://merge-csv-chi.vercel.app/" target="_blank" class="btn btn-sm btn-outline">Merge CSV App</a>
      </div>
    </header>

    <main class="mx-auto w-full px-4 py-6">

      <!-- ===== Stepper ===== -->
      <div class="flex items-center gap-2 mb-6 flex-wrap">
        <div v-for="(s, i) in steps" :key="s.num" class="flex items-center gap-2">
          <button @click="goToStep(s.num)" :class="stepClass(s.num)"
            class="flex items-center gap-2 px-3 py-2 rounded-lg border text-sm font-medium transition-colors">
            <span class="w-6 h-6 rounded-full flex items-center justify-center text-xs font-bold"
              :class="stepCircleClass(s.num)">{{ s.num }}</span>
            {{ s.label }}
          </button>
          <span v-if="i < steps.length - 1" class="text-gray-300">›</span>
        </div>
      </div>

      <!-- ===== Step 1: Upload ===== -->
      <section v-if="activeStep === 1" class="bg-white rounded-xl border border-gray-200 p-8">
        <div class="max-w-xl mx-auto text-center">
          <div class="w-16 h-16 bg-blue-50 rounded-full flex items-center justify-center mx-auto mb-4">
            <span class="text-3xl">📄</span>
          </div>
          <h2 class="text-2xl font-semibold mb-2">Upload your CSV</h2>
          <p class="text-gray-600 mb-6">Select a product CSV file to begin. Package quantities will be auto-detected
            from product titles.</p>

          <label class="block cursor-pointer">
            <input type="file" accept=".csv" @change="onFileChange" class="hidden" />
            <span class="btn btn-primary btn-lg w-full">Choose CSV File</span>
          </label>
          <p class="text-xs text-gray-400 mt-3">Supports standard Amazon wholesale CSV exports</p>
        </div>
      </section>

      <!-- ===== Step 2: Process ===== -->
      <section v-if="activeStep === 2" class="space-y-4">
        <div class="bg-white rounded-xl border border-gray-200 p-6">
          <div class="flex items-center justify-between flex-wrap gap-4">
            <div>
              <h2 class="text-xl font-semibold">Ready to process</h2>
              <p class="text-gray-600 text-sm mt-1">
                <span class="font-medium text-gray-800">{{ fileName }}</span> — {{ csvData.length }} rows loaded
              </p>
            </div>
            <button @click="startProcessing" :disabled="isProcessing" class="btn btn-primary btn-lg gap-2">
              <span v-if="!isProcessing">▶</span>
              <span v-else class="loading loading-spinner loading-sm"></span>
              {{ isProcessing ? 'Processing…' : 'Detect Package Quantities' }}
            </button>
          </div>
        </div>

        <!-- Progress -->
        <div v-if="isProcessing || isProcessed" class="bg-white rounded-xl border border-gray-200 p-6">
          <div class="flex items-center justify-between mb-3">
            <h3 class="font-semibold">{{ isProcessing ? 'Processing…' : 'Processing complete' }}</h3>
            <button v-if="isProcessing" @click="cancelProcessingRun"
              class="btn btn-sm btn-outline btn-error">Cancel</button>
          </div>
          <progress class="progress progress-primary w-full h-3" :value="processedCount"
            :max="rowsToProcess.length"></progress>
          <p class="text-sm text-gray-600 mt-2">{{ processedCount }} / {{ rowsToProcess.length }} rows ({{ percentDone
            }}%)</p>
        </div>
      </section>

      <!-- ===== Step 3 & 4: Review ===== -->
      <section v-if="activeStep >= 3" class="space-y-4">
        <!-- Stats bar -->
        <div class="grid grid-cols-2 md:grid-cols-4 gap-3">
          <div class="bg-white rounded-xl border border-gray-200 p-4">
            <div class="text-2xl font-bold text-gray-900">{{ stats.total }}</div>
            <div class="text-sm text-gray-500">Total rows</div>
          </div>
          <div class="bg-white rounded-xl border border-gray-200 p-4">
            <div class="text-2xl font-bold text-blue-600">{{ stats.auto }}</div>
            <div class="text-sm text-gray-500">Auto-detected</div>
          </div>
          <div class="bg-white rounded-xl border border-gray-200 p-4">
            <div class="text-2xl font-bold text-amber-600">{{ stats.edited }}</div>
            <div class="text-sm text-gray-500">Manually edited</div>
          </div>
          <div class="bg-white rounded-xl border border-gray-200 p-4">
            <div class="text-2xl font-bold text-green-600">{{ stats.unchanged }}</div>
            <div class="text-sm text-gray-500">Unchanged</div>
          </div>
        </div>

        <!-- Review controls -->
        <div class="bg-white rounded-xl border border-gray-200 p-4">
          <div class="flex flex-wrap items-center gap-3">
            <!-- Global search -->
            <input v-model="globalSearch" @input="debounceApplyFilters"
              class="input input-bordered input-sm w-64 bg-white text-gray-900" placeholder="Search all columns…" />

            <!-- Status filter -->
            <select v-model="statusFilter" @change="applyFilters"
              class="select select-bordered select-sm bg-white text-gray-900">
              <option value="">All statuses</option>
              <option value="auto">Auto-detected</option>
              <option value="edited">Manually edited</option>
              <option value="unchanged">Unchanged</option>
            </select>

            <div class="flex-1"></div>

            <button @click="downloadFilteredCSV" class="btn btn-success btn-sm gap-2">
              <span>⬇</span> Download CSV
            </button>
          </div>

          <!-- Column toggles -->
          <div class="mt-3 pt-3 border-t border-gray-100">
            <div class="flex flex-wrap gap-2">
              <button v-for="col in columns" :key="col" @click="toggleColumn(col)"
                :class="visibleColumns[col] !== false ? 'bg-blue-50 text-blue-700 border-blue-200' : 'bg-white text-gray-500 border-gray-200'"
                class="px-2 py-1 rounded text-xs border">
                {{ col }}
              </button>
            </div>
          </div>
        </div>

        <!-- Table -->
        <div class="bg-white rounded-xl border border-gray-200 overflow-hidden">
          <div class="overflow-x-auto" style="max-height:65vh; overflow-y:auto;">
            <table class="table table-sm w-full">
              <thead class="sticky top-0 z-30 bg-gray-50">
                <tr>
                  <th
                    class="sticky left-0 z-40 bg-gray-50 w-16 min-w-16 max-w-16 text-center font-semibold text-gray-700">
                    Status</th>
                  <th v-for="col in displayColumns" :key="col"
                    :class="thColClass(col) + ' font-semibold text-gray-700'">
                    <div class="flex flex-col items-center gap-1 w-full">
                      <span @click="sortByColumn(col)"
                        class="cursor-pointer select-none inline-flex items-center gap-1 whitespace-nowrap">
                        {{ col }}
                        <span v-if="sortKey === col" class="text-blue-600">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                      </span>
                      <div v-if="isNumericColumn(col)" class="flex items-center gap-0.5">
                        <select v-model="filterOperators[col]" @change="applyFilters"
                          class="select select-xs select-bordered bg-white text-gray-900 w-10 px-0 text-center">
                          <option value=">">&gt;</option>
                          <option value="<">&lt;</option>
                          <option value="=">=</option>
                          <option value=">=">&gt;=</option>
                          <option value="<=">&lt;=</option>
                        </select>
                        <input v-model="filterValues[col]" @input="debounceApplyFilters"
                          class="input input-xs input-bordered bg-white text-gray-900 w-14" placeholder="val" />
                      </div>
                      <input v-else v-model="filterValues[col]" @input="debounceApplyFilters"
                        class="input input-xs input-bordered bg-white text-gray-900 w-24" placeholder="filter" />
                    </div>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, idx) in paginatedFilteredData" :key="idx" class="group hover:bg-blue-50">
                  <td class="sticky left-0 z-20 bg-white group-hover:bg-blue-50 w-16 min-w-16 max-w-16 text-center">
                    <span class="badge badge-sm" :class="statusBadgeClass(getRowStatus(row))">
                      {{ statusLabel(getRowStatus(row)) }}
                    </span>
                  </td>
                  <td v-for="col in displayColumns" :key="col" :class="tdColClass(col) + ' text-gray-700'">
                    <!-- Editable Package Quantity -->
                    <div v-if="col === packageQtyColumn" class="inline-flex items-center gap-1">
                      <div v-if="!isEditingCell(row, col)" @click.stop="enterEdit(row, col)"
                        class="px-2 py-0.5 rounded cursor-pointer select-none hover:bg-blue-100 hover:text-blue-700 font-medium text-center min-w-[2.5rem]"
                        :title="'Click to edit ' + col">
                        {{ formatPackageQuantity(row[col]) }}
                      </div>
                      <div v-else class="flex items-center gap-1">
                        <input ref="activeEditor" v-model="editValue" @keydown.enter.prevent="commitEdit()"
                          @keydown.esc.prevent="cancelEdit" @keydown.arrow-up.prevent="stepEdit(1)"
                          @keydown.arrow-down.prevent="stepEdit(-1)" @blur="commitEdit" type="text" inputmode="numeric"
                          class="input input-xs input-bordered w-20 text-center font-semibold bg-white text-gray-900" />
                        <button class="btn btn-ghost btn-xs text-green-600" @mousedown.prevent="commitEdit">✓</button>
                        <button class="btn btn-ghost btn-xs text-red-500" @mousedown.prevent="cancelEdit">✕</button>
                      </div>
                    </div>
                    <!-- Links -->
                    <a v-else-if="isLink(row[col])" :href="row[col]" target="_blank"
                      class="text-blue-600 underline truncate block max-w-[200px]">{{ row[col] }}</a>
                    <!-- Plain text (title wraps to show full text) -->
                    <span v-else class="break-words">{{ row[col] }}</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <div class="p-3 border-t border-gray-100 flex justify-center">
            <button class="btn btn-outline btn-sm" @click="showMoreRows" v-if="visibleRows < filteredData.length">
              Show more ({{ filteredData.length - visibleRows }} remaining)
            </button>
          </div>
        </div>
      </section>

      <!-- Error -->
      <div v-if="error" class="alert alert-error mt-4">
        <span>{{ error }}</span>
      </div>

    </main>
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
      batchSize: 50,
      filterTimeout: null,
      filterOperators: {},
      filterValues: {},
      filteredData: [],
      sortKey: "",
      sortOrder: "",
      visibleRows: 50,
      defaultVisibleRows: 50,
      config: {
        treatBareNumberPackAsMultipack: true,
        maxReasonablePackageQty: 200
      },
      originalBaseCosts: {},
      editing: { row: null, col: null },
      editValue: "",
      titleColumn: null,
      packageQtyColumn: null,
      costColumn: null,
      rowStatus: {},
      visibleColumns: {},
      globalSearch: "",
      statusFilter: "",
      activeStep: 1,
      fileName: "",
      steps: [
        { num: 1, label: "Upload" },
        { num: 2, label: "Process" },
        { num: 3, label: "Review" },
        { num: 4, label: "Export" }
      ]
    };
  },
  computed: {
    paginatedFilteredData() {
      return this.filteredData.slice(0, this.visibleRows);
    },
    percentDone() {
      if (!this.rowsToProcess.length) return 0;
      return ((this.processedCount / this.rowsToProcess.length) * 100).toFixed(1);
    },
    stats() {
      const total = this.csvData.length;
      const auto = Object.values(this.rowStatus).filter(s => s === 'auto').length;
      const edited = Object.values(this.rowStatus).filter(s => s === 'edited').length;
      const unchanged = Object.values(this.rowStatus).filter(s => s === 'unchanged').length;
      return { total, auto, edited, unchanged };
    },
    displayColumns() {
      const priority = [
        this.packageQtyColumn,
        this.titleColumn,
        this.costColumn,
        "Profit", "Margin", "ROI", "Sell Price", "Break Even Sell Price",
        "Bought In Past Month", "Est Sales", "Sales Rank", "Category", "Brand", "ASIN"
      ];
      const ordered = [];
      priority.forEach(c => {
        if (c && this.columns.includes(c) && !ordered.includes(c)) ordered.push(c);
      });
      this.columns.forEach(c => {
        if (!ordered.includes(c)) ordered.push(c);
      });
      return ordered.filter(c => this.visibleColumns[c] !== false);
    },
    hasData() { return this.csvData.length > 0; },
    isProcessed() { return this.processedCount > 0; },
    isComplete() {
      return this.processedCount === this.rowsToProcess.length && this.rowsToProcess.length > 0;
    }
  },
  methods: {
    stepClass(num) {
      const base = "border";
      if (num === this.activeStep) return base + " border-blue-500 bg-blue-50 text-blue-700";
      if (num < this.activeStep) return base + " border-green-300 bg-green-50 text-green-700";
      return base + " border-gray-200 text-gray-400";
    },
    stepCircleClass(num) {
      if (num === this.activeStep) return "bg-blue-600 text-white";
      if (num < this.activeStep) return "bg-green-500 text-white";
      return "bg-gray-200 text-gray-500";
    },
    statusBadgeClass(status) {
      return {
        auto: 'badge-info',
        edited: 'badge-warning',
        unchanged: 'badge-success',
        pending: 'badge-ghost'
      }[status] || 'badge-ghost';
    },
    thColClass(col) {
      if (col === this.packageQtyColumn) return 'sticky left-16 z-40 bg-gray-50 w-28 min-w-28 max-w-28 text-center';
      if (col === this.titleColumn) return 'sticky left-[176px] z-40 bg-gray-50 w-[360px] min-w-[360px] max-w-[360px]';
      return '';
    },
    tdColClass(col) {
      if (col === this.packageQtyColumn) return 'sticky left-16 z-20 bg-white group-hover:bg-blue-50 w-28 min-w-28 max-w-28 whitespace-nowrap';
      if (col === this.titleColumn) return 'sticky left-[176px] z-20 bg-white group-hover:bg-blue-50 w-[360px] min-w-[360px] max-w-[360px] whitespace-normal align-top';
      return 'whitespace-nowrap';
    },
    detectColumns() {
      this.titleColumn = this.columns.find(col =>
        col.toLowerCase().includes('title') ||
        col.toLowerCase().includes('description') ||
        col.toLowerCase().includes('product name')
      ) || 'Title';

      this.packageQtyColumn = this.columns.find(col =>
        col.toLowerCase().includes('package qty') ||
        col.toLowerCase().includes('package quantity') ||
        col.toLowerCase().includes('qty') ||
        col.toLowerCase().includes('quantity')
      ) || 'Package Quantity';

      this.costColumn = this.columns.find(col =>
        col.toLowerCase().includes('cost') ||
        col.toLowerCase().includes('unit cost') ||
        col.toLowerCase().includes('price')
      ) || 'Cost';
    },

    recalcRow(row, originalQtyBeforeEdit = null) {
      const newQty = this.parseNumeric(row[this.packageQtyColumn]) || 1;
      const rowId = row["ASIN"] || row["Product ID"] || JSON.stringify(row).slice(0, 50);

      if (!this.originalBaseCosts) this.originalBaseCosts = {};

      if (!this.originalBaseCosts[rowId]) {
        const originalCost = this.parseNumeric(row[this.costColumn]) || 0;
        const originalQty = originalQtyBeforeEdit || newQty || 1;
        this.originalBaseCosts[rowId] = originalQty > 0 ? originalCost / originalQty : originalCost;
      }

      const baseCostPerUnit = this.originalBaseCosts[rowId];
      const sellPrice = this.parseNumeric(row["Sell Price"]) || 0;
      const profit = this.parseNumeric(row["Profit"]) || 0;
      const vat = this.parseNumeric(row["VAT $"]) || 0;
      const inbound = this.parseNumeric(row["Inbound Shipping Estimate"]) || 0;

      const totalfees = sellPrice - profit - baseCostPerUnit;
      const totalCost = baseCostPerUnit * newQty;
      const newProfit = sellPrice - (totalCost + vat + inbound) - totalfees;

      const margin = sellPrice ? (newProfit / sellPrice) * 100 : 0;
      const roi = totalCost ? (newProfit / totalCost) * 100 : 0;
      const breakEvenSellPrice = totalCost + vat + inbound;

      row[this.costColumn] = this.formatCurrency(totalCost);
      row["Profit"] = this.formatCurrency(newProfit);
      row["Margin"] = margin.toFixed(2) + "%";
      row["ROI"] = roi.toFixed(2) + "%";
      row["Break Even Sell Price"] = this.formatCurrency(breakEvenSellPrice);
    },

    formatCurrency(value) {
      const abs = Math.abs(value);
      const formatted = abs.toFixed(2);
      const sign = value < 0 ? '-' : '';
      return `${sign}$${formatted}`;
    },

    async onFileChange(e) {
      this.error = "";
      const file = e.target.files[0];
      if (!file) return;

      this.fileName = file.name;
      this.originalBaseCosts = {};
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
      this.rowStatus = {};
      this.globalSearch = "";
      this.statusFilter = "";
      this.cancelEdit();
      this.cancelProcessing = false;
      this.isProcessing = false;
      this.activeStep = 2;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          this.csvData = results.data;
          this.columns = results.meta.fields;
          this.detectColumns();

          const requiredColumns = [this.packageQtyColumn, "Unit Count"];
          requiredColumns.forEach(col => {
            if (!this.columns.includes(col)) {
              this.columns.push(col);
              this.csvData.forEach(r => {
                if (r[col] === undefined) r[col] = "";
              });
            }
          });

          this.csvData.forEach(row => {
            if (row[this.packageQtyColumn] === "" || row[this.packageQtyColumn] === undefined || row[this.packageQtyColumn] === null) {
              row[this.packageQtyColumn] = 1;
            }
          });

          this.rowsToProcess = this.csvData.filter((row) => row[this.titleColumn]);

          this.columns.forEach(col => {
            this.filterOperators[col] = this.isNumericColumn(col) ? '>' : '';
            this.filterValues[col] = '';
            this.visibleColumns[col] = true;
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
      const HIGH_SANITY_LIMIT = 50;

      const packOfRegex = /\bpack\s+of\s+(\d+)\b/i;
      const packOfMatch = title.match(packOfRegex);
      if (packOfMatch) {
        const n = +packOfMatch[1];
        if (n > 0 && n <= MAX_QTY) multiPackIndicators.push({ value: n, type: "pack_of" });
      }

      const caseOfRegex = /\b(case|carton|ctn)\s+of\s+(\d+)\b/i;
      const caseOfMatch = title.match(caseOfRegex);
      if (caseOfMatch) {
        const n = +caseOfMatch[2];
        if (n > 0 && n <= MAX_QTY) multiPackIndicators.push({ value: n, type: "case_of" });
      }

      const nPackRegex = /\b(\d+)\s*-?\s*pack(s)?\b/gi;
      let nMatch;
      while ((nMatch = nPackRegex.exec(title)) !== null) {
        const n = +nMatch[1];
        if (n > 0 && n <= MAX_QTY) multiPackIndicators.push({ value: n, type: "n_pack" });
      }

      const wordPackRegex = new RegExp("\\b(" + Object.keys(multipackWordNumbers).join("|") + ")\\s+pack\\b", "i");
      const wordPackMatch = title.match(wordPackRegex);
      if (wordPackMatch) {
        const val = multipackWordNumbers[wordPackMatch[1].toLowerCase()];
        if (val > 0 && val <= MAX_QTY) multiPackIndicators.push({ value: val, type: "word_pack" });
      }

      const resolutionContextWords = /(display|screen|monitor|resolution|touchscreen|laptop|notebook|camera|pixel|px)\b/;
      const resolutionLike = /\b(\d{3,5})\s*[x×]\s*(\d{3,5})\b/i;
      const isResolution = resolutionLike.test(lower) && resolutionContextWords.test(lower);

      const hasPartNumberMarker = partNumberMarkers.some(m => lower.includes(m));

      const packXPattern = /\b(\d+)\s*[x×]\s*(\d+)(?:\s*(count|ct|pack|packs|pk|pks))?\b/gi;
      if (!isResolution) {
        let xp;
        while ((xp = packXPattern.exec(lower)) !== null) {
          const outer = +xp[1];
          const inner = +xp[2];
          const suffix = xp[3];

          if (outer >= 1000 || inner >= 1000) continue;
          if (hasPartNumberMarker && (outer > 50 || inner > 50) && !suffix) continue;
          if ((outer >= 200 && inner >= 200) && !suffix) continue;

          if (outer > 0 && outer <= MAX_QTY) {
            multiPackIndicators.push({ value: outer, type: "x_pattern" });
            if (suffix && inner > 0) unitCountCandidates.push({ value: inner });
          }
        }
      }

      const unitRegex = new RegExp("\\b(\\d{1,5})\\s*(" + unitContentWords.join("|") + ")\\b", "gi");
      let uc;
      while ((uc = unitRegex.exec(lower)) !== null) {
        const val = +uc[1];
        if (val > 0) unitCountCandidates.push({ value: val });
      }

      let packageQuantity = 1;
      let unitCount = null;

      if (multiPackIndicators.length) {
        const priority = { pack_of: 1, case_of: 2, x_pattern: 3, n_pack: 4, word_pack: 5 };
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
      } else if (unitCountCandidates.length) {
        unitCountCandidates.sort((a, b) => b.value - a.value);
        packageQuantity = unitCountCandidates[0].value;
      }

      if (unitCount == null && unitCountCandidates.length) {
        unitCountCandidates.sort((a, b) => b.value - a.value);
        unitCount = unitCountCandidates[0].value;
      }

      if (packageQuantity > HIGH_SANITY_LIMIT &&
        !['pack_of', 'case_of', 'n_pack', 'word_pack'].includes(multiPackIndicators[0]?.type)) {
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
        this.rowsToProcess = this.csvData.filter(r => r[this.titleColumn]);
        if (!this.rowsToProcess.length) return;
      }
      this.error = "";
      this.cancelProcessing = false;
      this.isProcessing = true;
      this.activeStep = 3;
      if (this.processedCount === this.rowsToProcess.length) {
        this.processedCount = 0;
        this.regexProcessedCount = 0;
      }
      this.processNextBatch(this.processedCount);
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
      this.rowStatus = {};
      this.rowsToProcess = this.csvData.filter(r => r[this.titleColumn]);
      this.rowsToProcess.forEach(r => {
        r[this.packageQtyColumn] = "";
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
        this.isProcessing = false;
        this.filteredData = [...this.csvData];
        this.activeStep = 4;
        return;
      }

      const end = Math.min(startIndex + this.batchSize, this.rowsToProcess.length);
      const batch = this.rowsToProcess.slice(startIndex, end);

      batch.forEach(row => {
        try {
          const title = row[this.titleColumn];
          const { packageQuantity, unitCount } = this.classifyTitle(title);
          const originalQty = this.parseNumeric(row[this.packageQtyColumn]) || 1;

          row[this.packageQtyColumn] = packageQuantity;
          if (unitCount !== null) row["Unit Count"] = unitCount;

          this.recalcRow(row, originalQty);
          this.regexProcessedCount++;

          const rowId = row["ASIN"] || row["Product ID"] || JSON.stringify(row).slice(0, 50);
          this.rowStatus[rowId] = packageQuantity !== originalQty ? 'auto' : 'unchanged';
        } catch {
          const originalQty = this.parseNumeric(row[this.packageQtyColumn]) || 1;
          row[this.packageQtyColumn] = 1;
          this.recalcRow(row, originalQty);
        }
        this.processedCount++;
      });

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
        col.toLowerCase().includes("unit") || col.toLowerCase().includes("profit") ||
        col.toLowerCase().includes("margin") || col.toLowerCase().includes("roi") ||
        col.toLowerCase().includes("break even") || col.toLowerCase().includes("sales") ||
        col.toLowerCase().includes("rank") || col.toLowerCase().includes("vat") ||
        col.toLowerCase().includes("shipping") || col.toLowerCase().includes("weight") ||
        col.toLowerCase().includes("rate") || col.toLowerCase().includes("reviews") ||
        col.toLowerCase().includes("ratings") || col.toLowerCase().includes("offers")) return true;
      const sample = this.csvData.slice(0, 20).map(r => this.isNumericValue(r[col]));
      const numericSamples = sample.filter(v => v);
      return numericSamples.length >= Math.ceil(sample.length * 0.5);
    },

    // True only when the whole cell value is a number (not just "contains a digit"),
    // so text columns like Title/ASIN/Brand aren't misclassified as numeric.
    isNumericValue(val) {
      if (val === null || val === undefined) return false;
      const s = String(val).trim();
      if (s === '') return false;
      const cleaned = s.replace(/[\$,%()<>]/g, '').replace(/,/g, '').trim();
      return /\d/.test(cleaned) && /^-?\d*\.?\d*$/.test(cleaned);
    },

    debounceApplyFilters() {
      clearTimeout(this.filterTimeout);
      this.filterTimeout = setTimeout(() => {
        this.applyFilters();
      }, 250);
    },

    applyFilters() {
      const search = this.globalSearch.trim().toLowerCase();
      const status = this.statusFilter;
      this.filteredData = this.csvData.filter(row => {
        // Status filter
        if (status && this.getRowStatus(row) !== status) return false;
        // Global search
        if (search) {
          const match = this.columns.some(key => {
            const val = row[key];
            return val != null && String(val).toLowerCase().includes(search);
          });
          if (!match) return false;
        }
        // Per-column filters (operator + value)
        for (const key of this.columns) {
          const filterVal = this.filterValues[key];
          if (!filterVal) continue;
          const operator = this.filterOperators[key];
          const actual = row[key];
          if (this.isNumericColumn(key)) {
            const actualNum = this.parseNumeric(actual);
            const value = this.parseNumeric(filterVal);
            if (isNaN(value) || isNaN(actualNum)) return false;
            switch (operator) {
              case '>': if (!(actualNum > value)) return false; break;
              case '<': if (!(actualNum < value)) return false; break;
              case '=': if (!(actualNum === value)) return false; break;
              case '>=': if (!(actualNum >= value)) return false; break;
              case '<=': if (!(actualNum <= value)) return false; break;
              default: break;
            }
          } else {
            if (!String(actual ?? '').toLowerCase().includes(String(filterVal).toLowerCase())) return false;
          }
        }
        return true;
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

    toggleColumn(col) {
      this.visibleColumns[col] = this.visibleColumns[col] === false;
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
      const baseName = this.fileName || "products.csv";
      link.setAttribute("download", "filter-" + baseName);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

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

      const originalQty = this.parseNumeric(row[col]) || 1;
      const qty = this.sanitizeQuantity(this.editValue);
      row[col] = qty;

      this.recalcRow(row, originalQty);

      const rowId = row["ASIN"] || row["Product ID"] || JSON.stringify(row).slice(0, 50);
      this.rowStatus[rowId] = 'edited';

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

    getRowStatus(row) {
      const rowId = row["ASIN"] || row["Product ID"] || JSON.stringify(row).slice(0, 50);
      return this.rowStatus[rowId] || 'pending';
    },

    statusLabel(status) {
      return {
        auto: 'Auto',
        edited: 'Edited',
        unchanged: 'Same',
        pending: 'Pending'
      }[status] || status;
    },

    goToStep(step) {
      if (step === 2 && !this.hasData) return;
      if (step === 3 && !this.isProcessed) return;
      if (step === 4 && !this.isComplete) return;
      this.activeStep = step;
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
    }
  },
  mounted() {
    document.addEventListener("click", this.onOutsideClick);
  },
  beforeUnmount() {
    document.removeEventListener("click", this.onOutsideClick);
  }
};
</script>
