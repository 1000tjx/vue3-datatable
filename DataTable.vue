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
  ],
  setup() {},
  data() {
    return {
      sortCol: null,
      sortEquation: 0,
      currentPage: 0,
      focusedRow: null,
    };
  },
  methods: {
    updateSortCol(col) {
      console.log(this.maxPages);
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
  },
  computed: {
    pages() {
      let p = parseInt(this.data.length / this.getRowsPerPage);
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
          {this.currentPage + 1}
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
          return row[col.name];
        }
      };
    },
    dataFilter() {
      let rows = [...this.data].map((row, index) => {
        row._dataTableSortNumber = index;
        return row;
      });
      if (!this.sortCol || this.sortEquation === 0) {
      } else {
        if (this.sortEquation === 1) {
          rows = rows.sort((a, b) =>
            a[this.sortCol] > b[this.sortCol] ? 1 : -1
          );
        } else if (this.sortEquation === 2) {
          rows = rows.sort((a, b) =>
            a[this.sortCol] < b[this.sortCol] ? 1 : -1
          );
        }
      }
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
          return <i class="bi bi-sort-down-alt sort-icon"></i>;
        }
      };
    },
  },

  render() {
    let cols = this.cols;
    let rows = this.dataFilter;
    if (rows.length === 0) {
      return "No data to show.";
    }
    return (
      <div>
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
                      onClick={() => this.updateSortCol(col.name)}
                    >
                      <div>
                        <span>{col.title}</span>
                        <span>{this.renderSortEquation(col.name)}</span>
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
                      backgroundColor:
                        row._dataTableSortNumber === this.focusedRow
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
          <div id="rows-count">{this.getRowsCount} total row(s)</div>
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
