## excel-helper.js
```javascript
import XLSX from 'xlsx';
/**
 * 将文件下载到本地
 * @param {Blob, File} file
 * @param {String} fileName
 */
export function saveAs (file, fileName) {
  let tmpa = document.createElement('a');
  tmpa.download = fileName;
  tmpa.href = URL.createObjectURL(file);
  tmpa.click();
  setTimeout(function () {
    URL.revokeObjectURL(file);
  }, 100);
}
/**
 * String to ArrayBuffer, 字符串转换成字节流
 * @param {String} str
 */
export function s2ab (str) {
  let buf = new ArrayBuffer(str.length);
  let view = new Uint8Array(buf);
  for (let i = 0; i !== str.length; ++i) view[i] = str.charCodeAt(i) & 0xff;
  return buf;
}

function Workbook () {
  if (!(this instanceof Workbook)) return new Workbook();
  this.SheetNames = [];
  this.Sheets = {};
}

function datenum (v, date1904) {
  if (date1904) {
    v += 1462;
  }
  let epoch = Date.parse(v);
  return (epoch - new Date(Date.UTC(1899, 11, 30))) / (24 * 60 * 60 * 1000);
}

function sheetFromArrayOfArrays (data) {
  let ws = {};
  let range = { s: { c: 10000000, r: 10000000 }, e: { c: 0, r: 0 } };
  for (let R = 0; R !== data.length; ++R) {
    for (let C = 0; C !== data[R].length; ++C) {
      if (range.s.r > R) range.s.r = R;
      if (range.s.c > C) range.s.c = C;
      if (range.e.r < R) range.e.r = R;
      if (range.e.c < C) range.e.c = C;
      let cell = { v: data[R][C] };
      if (cell.v == null) continue;
      let cellRef = XLSX.utils.encode_cell({ c: C, r: R });

      if (typeof cell.v === 'number') cell.t = 'n';
      else if (typeof cell.v === 'boolean') cell.t = 'b';
      else if (cell.v instanceof Date) {
        cell.t = 'n';
        cell.z = XLSX.SSF._table[14];
        cell.v = datenum(cell.v);
      } else cell.t = 's';

      ws[cellRef] = cell;
    }
  }
  if (range.s.c < 10000000) ws['!ref'] = XLSX.utils.encode_range(range);
  return ws;
}

/**
 * 根据table导出excel, 只能导出一页
 * @param {dom} tableDom
 * @param {*} fileName
 */
export function export2ExcelByDom (tableDom, fileName) {
  const wopts = { bookType: 'xlsx', bookSST: false, type: 'binary' };
  const wb = { SheetNames: ['Sheet1'], Sheets: {}, Props: {} };
  wb.Sheets['Sheet1'] = XLSX.utils.table_to_sheet(tableDom);
  let tmpDown = new Blob([s2ab(XLSX.write(wb, wopts))], {
    type: 'application/octet-stream',
  });
  saveAs(tmpDown, fileName);
}

export function exportJson2Excel (th, jsonData, defaultTitle) {
  /* original data */

  let data = jsonData;
  data.unshift(th);
  let wsName = 'SheetJS';

  let wb = new Workbook();
  let ws = sheetFromArrayOfArrays(data);

  /* 设置worksheet每列的最大宽度 */
  const colWidth = data.map(row => row.map(val => {
    /* 先判断是否为null/undefined */
    if (val == null) {
      return { 'wch': 10 };
    } else if (val.toString().charCodeAt(0) > 255) {
      return { 'wch': val.toString().length * 2 };
    } else {
      return { 'wch': val.toString().length };
    }
  }));
  /* 以第一行为初始值 */
  let result = colWidth[0];
  for (let i = 1; i < colWidth.length; i++) {
    for (let j = 0; j < colWidth[i].length; j++) {
      if (result[j]['wch'] < colWidth[i][j]['wch']) {
        result[j]['wch'] = colWidth[i][j]['wch'];
      }
    }
  }
  ws['!cols'] = result;

  /* add worksheet to workbook */
  wb.SheetNames.push(wsName);
  wb.Sheets[wsName] = ws;

  let wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: false, type: 'binary' });
  let title = defaultTitle || 'excel-list';
  saveAs(new Blob([s2ab(wbout)], { type: 'application/octet-stream' }), title + '.xlsx');
}
```
