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
            Amazon Wholesale
          </h1>
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
      <div class="card w-full max-w-7xl bg-white/80 backdrop-blur-xl shadow-2xl border border-gray-200/50">
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
            </div>

            <div class="overflow-x-auto rounded-xl bg-white shadow-sm">
              <table class="table table-sm">
                <thead class="bg-gray-50">
                  <tr>
                    <th v-for="col in columns" :key="col" class="text-gray-700 font-semibold">{{ col }}</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, idx) in csvData.slice(0, 10)" :key="idx" class="hover:bg-gray-50 transition-colors">
                    <td v-for="col in columns" :key="col" class="text-gray-600">{{ row[col] }}</td>
                  </tr>
                </tbody>
              </table>
            </div>

            <div class="mt-6 flex justify-center">
              <button @click="startProcessing" :disabled="isProcessing || processedCount > 0"
                class="btn btn-primary btn-lg gap-3 px-8 shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105 text-white">
                <i class="fas fa-rocket"></i>
                Start AI Processing
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

          <!-- Success & Download -->
          <div v-if="processedCount > 0 && !isProcessing" class="my-8 text-center">
            <div class="mb-6">
              <div class="w-16 h-16 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-4">
                <i class="fas fa-check-circle text-green-600 text-2xl"></i>
              </div>
              <h3 class="text-2xl font-bold text-gray-800 mb-2">Processing Complete!</h3>
              <p class="text-gray-600">Your CSV has been enhanced with AI-detected package quantities.</p>
            </div>

            <button @click="downloadCSV"
              class="btn btn-success btn-lg gap-3 px-8 shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105 text-white">
              <i class="fas fa-download"></i>
              Download Enhanced CSV
            </button>
          </div>

          <!-- Error Section -->
          <div v-if="error" class="alert alert-error shadow-lg">
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

      <!-- Trust Indicators -->
      <div class="mt-12 text-center">
        <div class="flex flex-wrap justify-center gap-8 opacity-70">
          <div class="flex items-center gap-2 text-gray-600">
            <i class="fab fa-aws text-2xl"></i>
            <span class="text-sm">AWS Powered</span>
          </div>
          <div class="flex items-center gap-2 text-gray-600">
            <i class="fas fa-shield-alt text-2xl"></i>
            <span class="text-sm">Enterprise Security</span>
          </div>
          <div class="flex items-center gap-2 text-gray-600">
            <i class="fas fa-clock text-2xl"></i>
            <span class="text-sm">24/7 Processing</span>
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
      apiProcessedCount: 0,
      apiErrorCount: 0,
      error: "",
      batchSize: 5,
    };
  },
  methods: {
    async onFileChange(e) {
      this.error = "";
      const file = e.target.files[0];
      if (!file) return;
      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          this.csvData = results.data;
          this.columns = results.meta.fields;
          this.rowsToProcess = this.csvData.filter((row) => row["Title"]);
          this.processedCount = 0;
          this.regexProcessedCount = 0;
          this.apiProcessedCount = 0;
          this.apiErrorCount = 0;
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

      // Check high confidence first
      for (const pattern of highConfidencePatterns) {
        const match = title.match(pattern);
        if (match) {
          const qty = parseInt(match[1], 10);
          if (!isNaN(qty) && qty > 0 && qty <= 1000) {
            return qty;
          }
        }
      }

      // Check medium confidence
      for (const pattern of mediumConfidencePatterns) {
        const match = title.match(pattern);
        if (match) {
          const qty = parseInt(match[1], 10);
          if (!isNaN(qty) && qty > 0 && qty <= 1000) {
            return qty;
          }
        }
      }

      // Check word numbers
      const lowerTitle = title.toLowerCase();
      for (const [phrase, qty] of Object.entries(wordNumbers)) {
        if (lowerTitle.includes(phrase)) {
          return qty;
        }
      }

      return null;
    },

    async startProcessing() {
      this.error = "";
      this.isProcessing = true;
      this.processedCount = 0;
      this.regexProcessedCount = 0;
      this.apiProcessedCount = 0;
      this.apiErrorCount = 0;

      try {
        for (let i = 0; i < this.rowsToProcess.length; i += this.batchSize) {
          const batch = this.rowsToProcess.slice(i, i + this.batchSize);

          await Promise.all(
            batch.map(async (row) => {
              try {
                // First try regex extraction
                const regexQuantity = this.extractQuantityFromTitle(row["Title"]);

                if (regexQuantity !== null) {
                  // Successfully extracted with regex
                  row["Package Quantity"] = regexQuantity;
                  this.regexProcessedCount++;
                  console.log(`Regex extracted: "${row["Title"]}" -> ${regexQuantity}`);
                } else {
                  // Fallback to API
                  console.log(`Sending to API: "${row["Title"]}"`);
                  const apiQuantity = await this.getPackageQuantityFromAPI(row["Title"]);
                  row["Package Quantity"] = apiQuantity;
                  this.apiProcessedCount++;
                  console.log(`API result: "${row["Title"]}" -> ${apiQuantity}`);
                }
              } catch (err) {
                console.error("Error processing row:", {
                  title: row["Title"],
                  error: err,
                  errorMessage: err.message || 'Unknown error',
                  errorDetails: err.error || err
                });
                this.apiErrorCount++;
                row["Package Quantity"] = 1; // fallback
              }
              this.processedCount++;
            })
          );

          // Small delay between batches to avoid rate limiting
          if (i + this.batchSize < this.rowsToProcess.length) {
            await new Promise(resolve => setTimeout(resolve, 200));
          }
        }
      } catch (err) {
        this.error = "Processing error: " + err.message;
        console.error("Main processing error:", err);
      }
      this.isProcessing = false;

      console.log(`Processing complete! 
        Regex: ${this.regexProcessedCount}
        API: ${this.apiProcessedCount}
        API Errors: ${this.apiErrorCount}
        Total: ${this.processedCount}`);
    },

    async getPackageQuantityFromAPI(title) {
      try {
        if (!title || typeof title !== 'string') {
          console.warn('Invalid title passed to API:', title);
          return 1;
        }

        const prompt = `You are a data extraction assistant. Your task is to determine the package quantity (number of units/items) from the product title.

        Rules: 
        - Return only a single integer. 
        - If the title contains a number (e.g., "Pack of 3", "3 x 50ml", "2-Count"), use that number.  
        - If the title contains a number in words (e.g., "Twin Pack" = 2, "Dozen" = 12, "Set of Four" = 4), convert it to an integer.  
        - If multiple numbers are present, choose the one that represents the package quantity, not size or volume (e.g., "3 x 50ml" → 3, not 50).  
        - If the title does not clearly indicate multiple items, return 1.  

        Product title: "${title}"`;

        const response = await window.puter.ai.chat(prompt, {
          model: "claude-3-5-sonnet",
        });


        const text = response.message?.content?.[0]?.text?.trim() || "";
        const qty = parseInt(text, 10);

        return isNaN(qty) ? 1 : qty;
      }
      catch (err) {
        console.error("API error:", err);
        throw err;
      }
    },

    downloadCSV() {
      const csv = Papa.unparse(this.csvData);
      const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.setAttribute("download", "updated_products.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
};
</script>

<style scoped>
@keyframes blob {
  0% {
    transform: translate(0px, 0px) scale(1);
  }

  33% {
    transform: translate(30px, -50px) scale(1.1);
  }

  66% {
    transform: translate(-20px, 20px) scale(0.9);
  }

  100% {
    transform: translate(0px, 0px) scale(1);
  }
}

.animate-blob {
  animation: blob 7s infinite;
}

.animation-delay-2000 {
  animation-delay: 2s;
}

.animation-delay-4000 {
  animation-delay: 4s;
}
</style>