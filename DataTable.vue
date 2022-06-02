<script>
export default {
  props: [
    "data",
    "cols",
    "focusedRowColor",
    "rowsPerPage",
    "rowsCount",
    "onPageNext",
    "onPagePrev",
    "serverHandle",
    "onSearch",
    "showToggleAllFilters",
    "showSearchInput",
  ],
  setup() {},
  data() {
    return {
      sortCol: null,
      sortEquation: 0,
      currentPage: 0,
      focusedRow: null,
      colsToSearch: [...this.cols],
      query: "",
    };
  },
  methods: {
    getDeepValue(row, colName, forSort = false) {
      let names = colName.split(".");
      let finalVal = "";
      if (names.length === 1) finalVal = row[names[0]];

      finalVal = row;
      for (let name of names) {
        finalVal = finalVal[name];
      }
      if (!isNaN(Number(finalVal)) && forSort) {
        finalVal = Number(finalVal);
      }
      return finalVal;
    },
    updateSortCol(col) {
      if (col !== this.sortCol) {
        this.sortEquation = 1;
      } else {
        this.sortEquation = (this.sortEquation + 1) % 3;
      }
      this.sortCol = col;
    },
    nextPage() {
      this.currentPage = Math.min(++this.currentPage, this.pages);
      if (this.onPageNext) {
        this.onPageNext(this.currentPage);
      }
    },
    prevPage() {
      this.currentPage = Math.max(--this.currentPage, 0);
      if (this.onPagePrev) {
        this.onPagePrev(this.currentPage);
      }
    },
    addToSearchCol(col) {
      if (!this.colsToSearch.find((c) => c.name === col.name)) {
        this.colsToSearch.push(col);
      }
    },
    removeToSearchCol(col) {
      this.colsToSearch = this.colsToSearch.filter((c) => c.name !== col.name);
    },
    toggleAllFilters() {
      if (this.colsToSearch.length < this.cols.length) {
        this.colsToSearch = [...this.cols];
      } else {
        this.colsToSearch = [];
      }
    },
  },
  computed: {
    pages() {
      let p = parseInt(
        (this.rowsCount ?? this.data.length) / this.getRowsPerPage
      );
      if (this.data.length % this.getRowsPerPage > 0) ++p;
      return p;
    },
    getRowsCount() {
      return this.rowsCount ?? this.data.length;
    },
    getRowsPerPage() {
      return this.rowsPerPage || 2;
    },
    renderPaginationButtons() {
      let pageScaler = Math.max(this.pages - 1, 0);
      let buttons = [
        <button
          type="button"
          class="btn btn-dark btn-sm"
          onClick={() => this.prevPage()}
          disabled={this.currentPage == 0}
        >
          <i class="bi bi-arrow-left-short"></i>
        </button>,
        <button type="button" class="btn btn-dark btn-sm" disabled>
          {this.currentPage + 1} / {this.pages}
        </button>,
        <button
          type="button"
          class="btn btn-dark btn-sm"
          onClick={() => this.nextPage()}
          disabled={this.currentPage == pageScaler}
        >
          <i class="bi bi-arrow-right-short"></i>
        </button>,
      ];

      return buttons;
    },
    renderRow: function () {
      return (row, col) => {
        if (col.render) {
          return col.render(row);
        } else {
          return this.getDeepValue(row, col.name);
        }
      };
    },
    queryFilter() {
      return (rows) => {
        if (!this.onSearch) {
          if (this.colsToSearch.length === 0) return rows;
          return rows.filter((row) => {
            return this.colsToSearch.some((c) => {
              if (!c) return false;
              return this.getDeepValue(row, c.name)
                ?.toString()
                .toLowerCase()
                .includes(this.query.toLowerCase());
            });
          });
        } else {
          this.onSearch(this.query, this.colsToSearch);
          return rows;
        }
      };
    },
    dataFilter() {
      let rows = [...this.data].map((row, index) => {
        row._dataTableSortNumber = index;
        return row;
      });
      // search
      rows = this.queryFilter(rows);
      // sorting
      if (!this.sortCol || this.sortEquation === 0) {
      } else {
        if (this.sortEquation === 1) {
          rows = rows.sort((a, b) =>
            this.getDeepValue(a, this.sortCol, true) >
            this.getDeepValue(b, this.sortCol, true)
              ? 1
              : -1
          );
        } else if (this.sortEquation === 2) {
          rows = rows.sort((a, b) =>
            this.getDeepValue(a, this.sortCol, true) <
            this.getDeepValue(b, this.sortCol, true)
              ? 1
              : -1
          );
        }
      }
      // handle pagination
      if (!this.serverHandle) {
        rows = rows.slice(
          this.currentPage * this.getRowsPerPage,
          this.currentPage * this.getRowsPerPage + this.getRowsPerPage
        );
      }
      return rows;
    },
    renderSortEquation() {
      return (col) => {
        if (this.sortCol !== col) return <span class="sort-icon">=</span>;
        if (this.sortEquation === 0) {
          return "=";
        } else if (this.sortEquation === 1) {
          //return <i class="bi bi-arrow-up-short"></i>;
          return <i class="bi bi-sort-up-alt sort-icon"></i>;
        } else if (this.sortEquation === 2) {
          //return <i class="bi bi-arrow-down-short"></i>;
          return <i class="bi bi-sort-down sort-icon"></i>;
        }
      };
    },
    renderColsToSearch() {
      let items = this.colsToSearch.map((col, index) => {
        return (
          <button
            type="button"
            class="btn btn-sm btn-success"
            key={index}
            onClick={() => this.removeToSearchCol(col)}
          >
            <span>
              <i class="bi bi-x-lg"></i>
            </span>
            <span> {col.title} </span>
          </button>
        );
      });
      return items;
    },
    renderFilterIcon() {
      if (!this.showSearchInput) {
        return () => "";
      }
      return (col) => {
        if (this.colsToSearch.find((c) => c.name === col.name)) {
          return (
            <i
              title="Add to filter"
              class="bi bi-funnel-fill"
              onClick={() => this.removeToSearchCol(col)}
            ></i>
          );
        }
        return (
          <i
            title="Add to filter"
            class="bi bi-funnel"
            onClick={() => this.addToSearchCol(col)}
          ></i>
        );
      };
    },
  },
  render() {
    let cols = this.cols;
    let rows = this.dataFilter;
    if (rows.length === 0 && !this.query?.trim()) {
      return <p>No data to show.</p>;
    }
    return (
      <div>
        {!this.showSearchInput ? null : (
          <div>
            <div class="input-group input-group-sm mb-3">
              {this.showToggleAllFilters ? (
                <div
                  class="input-group-prepend"
                  style={{ cursor: "pointer" }}
                  onClick={this.toggleAllFilters}
                >
                  <span class="input-group-text">
                    {this.colsToSearch.length > 0 ? (
                      <i class="bi bi-funnel-fill"></i>
                    ) : (
                      <i class="bi bi-funnel"></i>
                    )}
                  </span>
                </div>
              ) : null}
              <div class="input-group-prepend">
                <span class="input-group-text" id="search">
                  <i class="bi bi-search"></i>
                </span>
              </div>
              <input
                onInput={(e) => (this.query = e.target.value)}
                value={this.query}
                placeholder="Search..."
                type="text"
                class="form-control"
                aria-label="search"
              />
            </div>
            <div class="btn-group">{this.renderColsToSearch}</div>
            <br />
            <br />
          </div>
        )}
        <div class="table-responsive">
          <table class="table table-sm table-bordered table-striped">
            <thead class="table-dark">
              <tr>
                {cols.map((col, colIndex) => {
                  return (
                    <th
                      scope="col"
                      key={colIndex}
                      style={{
                        minWidth: col.width ?? "auto",
                        width: col.width ?? "auto",
                      }}
                    >
                      <div>
                        <div
                          style={{ flex: 1 }}
                          onClick={() => this.updateSortCol(col.name)}
                        >
                          <span>{this.renderSortEquation(col.name)}</span>
                          <span> {col.title} </span>
                        </div>
                        <div>{this.renderFilterIcon(col)}</div>
                      </div>
                    </th>
                  );
                })}
              </tr>
            </thead>
            <tbody>
              {rows.map((row, rowIndex) => {
                return (
                  <tr
                    key={rowIndex}
                    style={{
                      backgroundColor: row.__bg_color__
                        ? row.__bg_color__
                        : row._dataTableSortNumber === this.focusedRow
                        ? this.focusedRowColor || "#e1ffde"
                        : "",
                    }}
                    onClick={() => (this.focusedRow = row._dataTableSortNumber)}
                  >
                    {cols.map((col, colIndex) => {
                      return <td key={colIndex}>{this.renderRow(row, col)}</td>;
                    })}
                  </tr>
                );
              })}
            </tbody>
          </table>
        </div>
        {/* pagination */}
        <div id="dt-pagination">
          <div id="pages-buttons">
            <div class="btn-group" role="group" aria-label="Pagination">
              {this.renderPaginationButtons}
            </div>
          </div>
          <div id="rows-count">{this.getRowsCount} = Total row(s)</div>
        </div>
      </div>
    );
  },
};
</script>

<style scoped>
#dt-pagination {
  display: flex;
  justify-content: space-between;
}
#rows-count {
  font-size: 0.85em;
}
table tbody {
  font-size: 0.85em;
}
button {
  font-size: 0.75em;
}

th {
  cursor: pointer;
  font-size: 0.9em;
}
.sort-icon {
  font-size: 1.2em;
}
th > div {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}
</style>
