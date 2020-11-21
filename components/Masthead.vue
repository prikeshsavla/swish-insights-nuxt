<template>
  <header class="masthead ">
    <div class="container">
      <div class="row">
        <div class="col-xl-9 mx-auto">
          <h1 class="mb-5">
            Build a landing page for your business or project and generate more
            leads!
          </h1>
        </div>
        <div class="col-md-10 col-lg-8 col-xl-7 mx-auto">
          <form @submit.prevent="run">
            <div class="form-row">
              <div class="col-12 col-md-9 mb-2 mb-md-0">
                <input
                  class="form-control form-control-lg"
                  placeholder="https://...."
                  v-model="url"
                />
              </div>
              <div class="col-12 col-md-3">
                <b-button
                  variant="primary"
                  size="lg"
                  block="true"
                  type="submit"
                  v-if="!loadingResults"
                >
                  Run
                </b-button>

                <b-button
                  variant="primary"
                  size="lg"
                  block="true"
                  disabled
                  v-else
                >
                  <b-spinner small></b-spinner>
                  <span class="sr-only">Loading...</span>
                </b-button>
              </div>
            </div>
          </form>
        </div>
      </div>
      <div class="col-xs-12" id="result"></div>

      <transition name="fade">
        <div
          class="result row bg-white"
          v-show="result != null && !loadingResults"
        >
          <div class="mx-auto col-lg-6">
            <h3>
              Result
              <small class="float-right">
                <b-button v-b-toggle.preview variant="primary" size="sm"
                  >Show JSON Response</b-button
                >
                <b-button
                  v-b-toggle.preview
                  variant="primary"
                  size="sm"
                  :href="reportLink"
                  target="_blank"
                  >View Report</b-button
                >
              </small>
            </h3>
            <div class="notification" id="result"></div>

            <table class="table table-sm mt-4">
              <tr v-for="(value, metric) in lighthouseMetrics">
                <th>{{ metric }}</th>
                <td>{{ value }}</td>
              </tr>
            </table>
          </div>
          <div class="col-12">
            <b-collapse id="preview" class="mt-2">
              <div class="preview-json overflow-auto">
                <pre v-html="result"></pre>
              </div>
            </b-collapse>
          </div>
        </div>
      </transition>
    </div>
  </header>
</template>

<script>
export default {
  data() {
    return {
      url: "",
      errorMessage: "",
      loadingResults: false,
      categories: [
        "accessibility",
        "best-practices",
        "performance",
        "pwa",
        "seo"
      ],
      result: null,
      reportLink: "",
      lighthouseMetrics: []
    };
  },
  methods: {
    titleize(sentence) {
      if (!sentence.split) return sentence;
      var _titleizeWord = function(string) {
          return string.charAt(0).toUpperCase() + string.slice(1).toLowerCase();
        },
        result = [];
      sentence.split(" ").forEach(function(w) {
        result.push(_titleizeWord(w));
      });
      return result.join(" ");
    },
    isValidUrl(str) {
      const regexp = /^(?:(?:http(s)?):\/\/)(?:(?!(?:10|127)(?:\.\d{1,3}){3})(?!(?:169\.254|192\.168)(?:\.\d{1,3}){2})(?!172\.(?:1[6-9]|2\d|3[0-1])(?:\.\d{1,3}){2})(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)(?:\.(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)*(?:\.(?:[a-z\u00a1-\uffff]{2,})))(?::\d{2,5})?(?:\/\S*)?$/;
      return regexp.test(str);
    },
    run() {
      this.loadingResults = true;
      this.result = null;
      this.lighthouseMetrics = [];
      if (!this.isValidUrl(this.url)) {
        this.loadingResults = false;
        this.errorMessage = "Please enter a valid URL (https://example.com)";
        return false;
      }
      this.errorMessage = "";
      const queryUrl = this.url.replace(/\/$/, "").toLowerCase();
      const url = this.setUpQuery(queryUrl);

      const that = this;

      fetch(url)
        .then(response => response.json())
        .then(json => {
          this.showInitialContent(json.id);
          const lighthouse = json.lighthouseResult;

          const lighthouseMetrics = {
            Performance: lighthouse.categories["performance"]
              ? lighthouse.categories["performance"].score * 100
              : null,
            Accessibility: lighthouse.categories["accessibility"]
              ? lighthouse.categories["accessibility"].score * 100
              : null,
            "Best-Practices": lighthouse.categories["best-practices"]
              ? lighthouse.categories["best-practices"].score * 100
              : null,
            PWA: lighthouse.categories["pwa"]
              ? lighthouse.categories["pwa"].score * 100
              : null,
            SEO: lighthouse.categories["seo"]
              ? lighthouse.categories["seo"].score * 100
              : null,
            "First Contentful Paint":
              lighthouse.audits["first-contentful-paint"].displayValue,
            "Speed Index": lighthouse.audits["speed-index"].displayValue,
            "Time To Interactive": lighthouse.audits.interactive.displayValue,
            "First Meaningful Paint":
              lighthouse.audits["first-meaningful-paint"].displayValue,
            "First CPU Idle": lighthouse.audits["first-cpu-idle"].displayValue,
            "Estimated Input Latency":
              lighthouse.audits["estimated-input-latency"].displayValue
          };

          this.loadingResults = false;
          this.lighthouseMetrics = lighthouseMetrics;
          this.result = this.syntaxHighlight(
            JSON.stringify(lighthouse, undefined, 2)
          );

          var data = JSON.stringify({
            description: "Lighthouse response ",
            public: true,
            files: {
              "file.lighthouse.json": {
                content: JSON.stringify(lighthouse, undefined, 2)
              }
            }
          });

          var config = {
            method: "post",
            url: "https://api.github.com/gists",
            headers: {
              Authorization: "Bearer b5c9a2b79235c633b63b2b15be986beb2286e780",
              "Content-Type": "application/json"
            },
            data: data
          };

          this.$axios(config)
            .then(response => {
              console.log(JSON.stringify(response.data.id));
              this.reportLink =
                "https://googlechrome.github.io/lighthouse/viewer/?gist=" +
                response.data.id;
              console.log(
                "https://googlechrome.github.io/lighthouse/viewer/?gist=" +
                  response.data.id
              );
            })
            .catch(function(error) {
              console.log(error);
            });

          /*
{
    "description": "Lighthouse response ",
    "public": true,
    "files": {
    "file.lighthouse.json": {
        "content": "{\"abc\": \"heelo\"}"
    }
    }
}
          */
        });
    },

    setUpQuery(queryUrl) {
      const api = "https://www.googleapis.com/pagespeedonline/v5/runPagespeed";

      const parameters = {
        url: encodeURIComponent(queryUrl),
        category: this.categories
      };
      let query = `${api}?key=AIzaSyDAnEMnj6igIF_WRqL5mgzumTKWgN_zojs`;
      for (const key in parameters) {
        if (!!parameters[key]) {
          if (Array.isArray(parameters[key])) {
            parameters[key].forEach(element => {
              query += `&${key}=${element}`;
            });
          } else {
            query += `&${key}=${parameters[key]}`;
          }
        }
      }
      return query;
    },

    showInitialContent(id) {
      document.getElementById("result").innerHTML = "";
      const page = document.createElement("p");
      page.innerHTML = `Page tested: <a href="${id}" target="_blank"><i class="fas fa-external-link-alt" aria-hidden="true"></i> ${id}</a>`;
      document.getElementById("result").appendChild(page);
    },
    syntaxHighlight(json) {
      json = json
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;");
      return json.replace(
        /("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g,
        function(match) {
          var cls = "number";
          if (/^"/.test(match)) {
            if (/:$/.test(match)) {
              cls = "key";
            } else {
              cls = "string";
            }
          } else if (/true|false/.test(match)) {
            cls = "boolean";
          } else if (/null/.test(match)) {
            cls = "null";
          }
          return '<span class="' + cls + '">' + match + "</span>";
        }
      );
    }
  }
};
</script>

<style lang="scss">
header.masthead {
  position: relative;
  @include background-cover;
  padding-top: 8rem;
  padding-bottom: 8rem;
  .overlay {
    position: absolute;
    background-color: $gray-900;
    height: 100%;
    width: 100%;
    top: 0;
    left: 0;
    opacity: 0.3;
  }
  h1 {
    font-size: 2rem;
  }
  @media (min-width: 768px) {
    padding-top: 12rem;
    padding-bottom: 12rem;
    h1 {
      font-size: 3rem;
    }
  }
}

.preview-json {
  max-height: 500px;
  pre {
    outline: 1px solid #ccc;
    padding: 5px;
    margin: 5px;
    white-space: pre-wrap;
  }

  .string {
    color: green;
  }

  .number {
    color: red;
  }

  .boolean {
    color: blue;
  }

  .null {
    color: magenta;
  }

  .key {
    color: brown;
  }
}
</style>
