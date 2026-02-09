# CATALOGO EXPORT-IMPORT v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER EXPORT/IMPORT DATI IN REACT/NEXT.JS

---

§ §1. EXPORT FORMATS COMPARISON

§ **TABELLA COMPARATIVA DETTAGLIATA**

| Format | Use Case | Library | Bundle Size | Styling | Performance | Best For | Limitations |
|--------|----------|---------|-------------|---------|-------------|----------|-------------|
| **CSV** | Data exchange, Excel import/export | Papa Parse | 19kB gzipped | None (plain text) | Excellent | Large datasets, simple data | No styling, no multiple sheets |
| **Excel (XLSX)** | Reports, formatted data | SheetJS (xlsx) | 126kB gzipped | Basic | Good | Excel compatibility, formulas | Advanced styling limited |
| **Excel (Advanced)** | Styled reports, complex layouts | ExcelJS | 187kB gzipped | Full (fonts, colors, borders) | Good | Professional reports | Larger bundle size |
| **PDF (Simple)** | Documents, invoices, tables | jsPDF + autoTable | 125kB gzipped | Basic tables | Good | Simple PDFs with tables | Complex layouts difficult |
| **PDF (Advanced)** | Complex documents, React templates | @react-pdf/renderer | 63kB gzipped | React-like styling | Medium | React developers, invoices | Limited browser preview |
| **JSON** | API responses, backup, data sync | Native | 0kB | None | Excellent | Data transfer, backup | No human-readable formatting |
| **XML** | Legacy systems, SOAP APIs | fast-xml-parser | 7kB gzipped | None | Good | Enterprise integrations | Verbose, outdated |

§ **DECISION MATRIX**

typescript
const EXPORT_FORMAT_RECOMMENDATIONS = {
  // Per tipo di dato:
  tableData: {
    small: 'CSV',
    large: 'CSV (streaming)',
    formatted: 'Excel (xlsx)',
    veryLarge: 'Background CSV export'
  },
  
  reports: {
    simple: 'PDF (jsPDF)',
    complex: 'PDF (react-pdf)',
    editable: 'Excel (ExcelJS)',
    interactive: 'Excel with formulas'
  },
  
  documents: {
    invoices: 'PDF (react-pdf)',
    receipts: 'PDF (jsPDF)',
    statements: 'PDF or Excel',
    labels: 'PDF with precise layout'
  },
  
  // Per ambiente:
  clientSide: {
    fast: 'CSV',
    formatted: 'Excel (xlsx)',
    printable: 'PDF (jsPDF)'
  },
  
  serverSide: {
    largeData: 'Streaming CSV',
    scheduled: 'Background job to S3',
    realtime: 'In-memory export'
  }
};

---

§ §2. CSV EXPORT

§ 2.1 PAPA PARSE EXPORT

typescript
// lib/export/csv.ts
import Papa from 'papaparse';

export interface CSVExportOptions {
  /** Filename for download (without .csv extension) */
  filename?: string;
  
  /** Column configuration */
  columns?: Array<{
    /** Data key in the object */
    key: string;
    /** Column header text */
    header: string;
    /** Value formatter function */
    formatter?: (value: any, row: any) => string;
    /** Include column in export (default: true) */
    include?: boolean;
  }>;
  
  /** CSV options */
  delimiter?: string;
  newline?: string;
  quoteChar?: string;
  escapeChar?: string;
  
  /** Data preprocessing */
  preprocess?: (data: any[]) => any[];
  
  /** BOM for Excel UTF-8 compatibility */
  bom?: boolean;
  
  /** Download options */
  download?: boolean;
}

export interface CSVExportResult {
  /** CSV data as string */
  csv: string;
  /** Number of rows exported */
  rowCount: number;
  /** Column headers */
  headers: string[];
  /** Filename used */
  filename: string;
  /** Blob URL if downloaded */
  blobUrl?: string;
}

/**
 * Export data to CSV using PapaParse
 */
export function exportToCSV<T extends Record<string, any>>(
  data: T[],
  options: CSVExportOptions = {}
): CSVExportResult {
  // Default options
  const {
    filename = `export-${new Date().toISOString().slice(0, 10)}`,
    columns,
    delimiter = ',',
    newline = '\n',
    quoteChar = '"',
    escapeChar = '"',
    preprocess,
    bom = true, // Add BOM for Excel UTF-8 compatibility
    download = true,
  } = options;
  
  // Preprocess data if needed
  const processedData = preprocess ? preprocess(data) : data;
  
  // Prepare data based on column configuration
  let exportData: any[];
  let headers: string[];
  
  if (columns) {
    // Filter and format based on columns configuration
    exportData = processedData.map(row => {
      const exportedRow: Record<string, any> = {};
      
      columns.forEach(col => {
        if (col.include !== false) {
          const value = row[col.key];
          exportedRow[col.header] = col.formatter 
            ? col.formatter(value, row) 
            : formatCSVValue(value);
        }
      });
      
      return exportedRow;
    });
    
    headers = columns
      .filter(col => col.include !== false)
      .map(col => col.header);
  } else {
    // Use all object keys
    exportData = processedData.map(row => {
      const exportedRow: Record<string, any> = {};
      
      Object.entries(row).forEach(([key, value]) => {
        exportedRow[key] = formatCSVValue(value);
      });
      
      return exportedRow;
    });
    
    headers = Object.keys(processedData[0] || {});
  }
  
  // Generate CSV using PapaParse
  const csv = Papa.unparse(exportData, {
    delimiter,
    newline,
    quoteChar,
    escapeChar,
    header: true,
    skipEmptyLines: true,
  });
  
  // Add BOM for Excel UTF-8 compatibility
  const csvWithBOM = bom ? '\uFEFF' + csv : csv;
  
  // Create result
  const result: CSVExportResult = {
    csv: csvWithBOM,
    rowCount: exportData.length,
    headers,
    filename: filename.endsWith('.csv') ? filename : `${filename}.csv`,
  };
  
  // Trigger download if requested
  if (download) {
    downloadCSV(result.csv, result.filename);
  }
  
  return result;
}

/**
 * Format value for CSV export
 */
function formatCSVValue(value: any): string {
  if (value == null || value === '') {
    return '';
  }
  
  if (value instanceof Date) {
    return value.toISOString();
  }
  
  if (typeof value === 'boolean') {
    return value ? 'TRUE' : 'FALSE';
  }
  
  if (typeof value === 'number') {
    return value.toString();
  }
  
  if (typeof value === 'object') {
    // Stringify objects, arrays
    return JSON.stringify(value);
  }
  
  // Handle strings that might contain delimiters or quotes
  const stringValue = String(value);
  
  // Check if we need to quote the value
  const needsQuotes = stringValue.includes(',') || 
                      stringValue.includes('\n') || 
                      stringValue.includes('\r') || 
                      stringValue.includes('"') ||
                      stringValue.trim() !== stringValue;
  
  if (needsQuotes) {
    // Escape existing quotes
    const escaped = stringValue.replace(/"/g, '""');
    return `"${escaped}"`;
  }
  
  return stringValue;
}

/**
 * Trigger CSV download
 */
function downloadCSV(csv: string, filename: string): void {
  // Create blob
  const blob = new Blob([csv], { 
    type: 'text/csv;charset=utf-8;' 
  });
  
  // Create download link
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  
  link.href = url;
  link.download = filename;
  link.style.display = 'none';
  
  // Trigger download
  document.body.appendChild(link);
  link.click();
  
  // Cleanup
  setTimeout(() => {
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }, 100);
}

/**
 * Export with column definitions example
 */
export const userExportColumns = [
  { key: 'id', header: 'ID' },
  { key: 'name', header: 'Full Name' },
  { key: 'email', header: 'Email Address' },
  { key: 'createdAt', header: 'Created Date', formatter: (val: Date) => val.toLocaleDateString() },
  { key: 'status', header: 'Status', formatter: (val: string) => val.toUpperCase() },
  { key: 'balance', header: 'Balance', formatter: (val: number) => `$${val.toFixed(2)}` },
];

/**
 * Stream CSV for large datasets (server-side)
 */
export async function streamCSV<T>(
  data: AsyncIterable<T> | T[],
  columns?: CSVExportOptions['columns'],
  writer: WritableStreamDefaultWriter = new WritableStream().getWriter()
): Promise<void> {
  // Implementation for streaming large datasets
  // This would be used server-side with a Response stream
}

// Example usage:
/*
const data = [
  { id: 1, name: 'John Doe', email: 'john@example.com', createdAt: new Date(), status: 'active', balance: 100.50 },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com', createdAt: new Date(), status: 'inactive', balance: 200.75 },
];

// Simple export
exportToCSV(data, { filename: 'users' });

// Advanced export with column configuration
exportToCSV(data, {
  filename: 'user-report',
  columns: userExportColumns,
  preprocess: (data) => data.filter(user => user.status === 'active'),
});
*/

§ 2.2 STREAMING CSV (LARGE DATASETS)

typescript
// lib/export/csv-stream.ts
import { TransformStream } from 'stream/web';

export interface CSVStreamOptions {
  /** Column headers */
  headers: string[];
  
  /** Batch size for processing */
  batchSize?: number;
  
  /** CSV delimiter */
  delimiter?: string;
  
  /** Include BOM for Excel */
  bom?: boolean;
}

/**
 * Create a ReadableStream that streams CSV data
 * Useful for large datasets that can't fit in memory
 */
export function createCSVStream<T>(
  dataSource: AsyncIterable<T> | (() => AsyncGenerator<T>),
  options: CSVStreamOptions
): ReadableStream<Uint8Array> {
  const {
    headers,
    batchSize = 1000,
    delimiter = ',',
    bom = true,
  } = options;
  
  // Helper to escape CSV values
  const escapeCSV = (value: any): string => {
    if (value == null) return '';
    
    const str = String(value);
    
    if (str.includes(delimiter) || str.includes('\n') || str.includes('\r') || str.includes('"')) {
      return `"${str.replace(/"/g, '""')}"`;
    }
    
    return str;
  };
  
  // Create headers row
  const headersRow = headers.map(escapeCSV).join(delimiter) + '\n';
  const bomBytes = bom ? new Uint8Array([0xEF, 0xBB, 0xBF]) : new Uint8Array(0);
  
  let isFirstChunk = true;
  
  // Return readable stream
  return new ReadableStream<Uint8Array>({
    async start(controller) {
      try {
        // Write BOM if requested
        if (bom) {
          controller.enqueue(bomBytes);
        }
        
        // Write headers
        controller.enqueue(new TextEncoder().encode(headersRow));
        
        // Get data iterator
        const iterator = typeof dataSource === 'function' 
          ? dataSource()
          : dataSource[Symbol.asyncIterator]();
        
        let batch: T[] = [];
        
        // Process data in batches
        while (true) {
          const result = await iterator.next();
          
          if (result.done) {
            // Process remaining batch
            if (batch.length > 0) {
              const batchCSV = batch.map(row => 
                headers.map(header => escapeCSV((row as any)[header])).join(delimiter)
              ).join('\n') + '\n';
              
              controller.enqueue(new TextEncoder().encode(batchCSV));
            }
            break;
          }
          
          batch.push(result.value);
          
          // Process batch when full
          if (batch.length >= batchSize) {
            const batchCSV = batch.map(row => 
              headers.map(header => escapeCSV((row as any)[header])).join(delimiter)
            ).join('\n') + '\n';
            
            controller.enqueue(new TextEncoder().encode(batchCSV));
            batch = [];
          }
        }
        
        controller.close();
      } catch (error) {
        controller.error(error);
      }
    },
    
    cancel() {
      // Cleanup if stream is cancelled
      console.log('CSV stream cancelled');
    },
  });
}

/**
 * Server-side API route for streaming CSV export
 */
// app/api/export/users/csv/route.ts
/*
import { NextRequest } from 'next/server';
import { createCSVStream } from '@/lib/export/csv-stream';
import { getUsersStream } from '@/lib/data/users';

export async function GET(request: NextRequest) {
  try {
    // Get query parameters
    const searchParams = request.nextUrl.searchParams;
    const fromDate = searchParams.get('from');
    const toDate = searchParams.get('to');
    
    // Create async generator for users data
    async function* userGenerator() {
      let page = 0;
      const limit = 1000;
      
      while (true) {
        const users = await getUsersStream(page, limit, { fromDate, toDate });
        
        if (users.length === 0) {
          break;
        }
        
        for (const user of users) {
          yield user;
        }
        
        page++;
      }
    }
    
    // Define CSV headers
    const headers = [
      'ID',
      'Name',
      'Email',
      'Created At',
      'Status',
      'Balance',
    ];
    
    // Create CSV stream
    const csvStream = createCSVStream(userGenerator, {
      headers,
      bom: true,
    });
    
    // Return streaming response
    return new Response(csvStream, {
      headers: {
        'Content-Type': 'text/csv; charset=utf-8',
        'Content-Disposition': `attachment; filename="users-export-${new Date().toISOString().slice(0, 10)}.csv"`,
        'Transfer-Encoding': 'chunked',
      },
    });
    
  } catch (error) {
    console.error('CSV export error:', error);
    return new Response(
      JSON.stringify({ error: 'Failed to generate CSV export' }),
      { 
        status: 500,
        headers: { 'Content-Type': 'application/json' },
      }
    );
  }
}
*/

/**
 * Client-side hook for streaming download
 */
// hooks/use-streaming-download.ts
export function useStreamingDownload() {
  const downloadStream = async (url: string, filename: string) => {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`Download failed: ${response.status}`);
      }
      
      if (!response.body) {
        throw new Error('ReadableStream not supported');
      }
      
      // Create stream reader
      const reader = response.body.getReader();
      const contentLength = response.headers.get('Content-Length');
      const total = contentLength ? parseInt(contentLength, 10) : 0;
      let received = 0;
      
      // Create blob chunks
      const chunks: Uint8Array[] = [];
      
      // Read stream
      while (true) {
        const { done, value } = await reader.read();
        
        if (done) {
          break;
        }
        
        chunks.push(value);
        received += value.length;
        
        // Update progress (optional)
        if (total > 0) {
          const percent = Math.round((received / total) * 100);
          console.log(`Download progress: ${percent}%`);
        }
      }
      
      // Combine chunks and create download
      const blob = new Blob(chunks);
      const downloadUrl = URL.createObjectURL(blob);
      
      const a = document.createElement('a');
      a.href = downloadUrl;
      a.download = filename;
      a.click();
      
      // Cleanup
      URL.revokeObjectURL(downloadUrl);
      
    } catch (error) {
      console.error('Streaming download failed:', error);
      throw error;
    }
  };
  
  return { downloadStream };
}

---

§ §3. EXCEL EXPORT

§ 3.1 XLSX LIBRARY (SHEETJS)

typescript
// lib/export/excel-xlsx.ts
import * as XLSX from 'xlsx';

export interface ExcelExportOptions {
  /** Filename (without .xlsx extension) */
  filename?: string;
  
  /** Sheet name */
  sheetName?: string;
  
  /** Column configuration */
  columns?: Array<{
    /** Data key */
    key: string;
    /** Column header */
    header: string;
    /** Column width in characters */
    width?: number;
    /** Cell formatting */
    format?: string; // Excel format string, e.g., '$#,##0.00', 'yyyy-mm-dd'
    /** Value formatter */
    formatter?: (value: any, row: any) => any;
  }>;
  
  /** Multiple sheets configuration */
  sheets?: Array<{
    name: string;
    data: any[];
    columns?: ExcelExportOptions['columns'];
  }>;
  
  /** Auto-fit columns */
  autoFitColumns?: boolean;
  
  /** Freeze header row */
  freezeHeader?: boolean;
  
  /** Styling options */
  styles?: {
    /** Header row style */
    header?: Partial<XLSX.CellStyle>;
    /** Data row style */
    data?: Partial<XLSX.CellStyle>;
    /** Alternating row colors */
    alternateRows?: boolean;
  };
}

export interface ExcelExportResult {
  /** Workbook object */
  workbook: XLSX.WorkBook;
  /** Generated file as ArrayBuffer */
  buffer: ArrayBuffer;
  /** Filename used */
  filename: string;
  /** Number of sheets */
  sheetCount: number;
  /** Total rows exported */
  totalRows: number;
}

/**
 * Export data to Excel using SheetJS (xlsx)
 */
export function exportToExcel<T extends Record<string, any>>(
  data: T[] | T[][],
  options: ExcelExportOptions = {}
): ExcelExportResult {
  const {
    filename = `export-${new Date().toISOString().slice(0, 10)}`,
    sheetName = 'Sheet1',
    columns,
    sheets,
    autoFitColumns = true,
    freezeHeader = true,
    styles = {},
  } = options;
  
  // Create new workbook
  const workbook = XLSX.utils.book_new();
  
  let totalRows = 0;
  
  // Handle single sheet or multiple sheets
  if (sheets) {
    // Multiple sheets
    sheets.forEach((sheetConfig, index) => {
      const { name, data: sheetData, columns: sheetColumns } = sheetConfig;
      
      const worksheet = createWorksheet(sheetData, {
        ...options,
        columns: sheetColumns || columns,
        sheetName: name,
        autoFitColumns,
        freezeHeader,
        styles,
      });
      
      XLSX.utils.book_append_sheet(workbook, worksheet, name || `Sheet${index + 1}`);
      totalRows += sheetData.length;
    });
  } else {
    // Single sheet
    const worksheet = createWorksheet(data as T[], {
      ...options,
      sheetName,
      autoFitColumns,
      freezeHeader,
      styles,
    });
    
    XLSX.utils.book_append_sheet(workbook, worksheet, sheetName);
    totalRows = (data as T[]).length;
  }
  
  // Generate buffer
  const buffer = XLSX.write(workbook, {
    type: 'array',
    bookType: 'xlsx',
    compression: true,
  });
  
  // Trigger download
  downloadExcel(buffer, filename);
  
  return {
    workbook,
    buffer,
    filename: filename.endsWith('.xlsx') ? filename : `${filename}.xlsx`,
    sheetCount: sheets ? sheets.length : 1,
    totalRows,
  };
}

/**
 * Create a worksheet with data
 */
function createWorksheet<T>(
  data: T[],
  options: {
    columns?: ExcelExportOptions['columns'];
    sheetName?: string;
    autoFitColumns: boolean;
    freezeHeader: boolean;
    styles: ExcelExportOptions['styles'];
  }
): XLSX.WorkSheet {
  const { columns, autoFitColumns, freezeHeader, styles } = options;
  
  let worksheetData: any[][];
  let headers: string[] = [];
  
  if (columns) {
    // Prepare headers
    headers = columns.map(col => col.header);
    
    // Prepare data with formatting
    worksheetData = data.map(row => {
      return columns.map(col => {
        const value = row[col.key as keyof T];
        return col.formatter ? col.formatter(value, row) : value;
      });
    });
  } else {
    // Use all object keys
    if (data.length > 0) {
      headers = Object.keys(data[0] as any);
      worksheetData = data.map(row => headers.map(header => (row as any)[header]));
    } else {
      headers = [];
      worksheetData = [];
    }
  }
  
  // Create worksheet with headers and data
  const worksheet = XLSX.utils.aoa_to_sheet([
    headers,
    ...worksheetData,
  ]);
  
  // Apply column widths if auto-fitting
  if (autoFitColumns && headers.length > 0) {
    const colWidths = headers.map((header, index) => {
      // Calculate max width based on header and data
      const headerLength = header.length;
      const dataLengths = worksheetData.map(row => {
        const cell = row[index];
        return cell ? String(cell).length : 0;
      });
      
      const maxDataLength = Math.max(...dataLengths);
      return Math.max(headerLength, maxDataLength) + 2; // Add padding
    });
    
    worksheet['!cols'] = colWidths.map(width => ({ wch: width }));
  } else if (columns) {
    // Use configured widths
    worksheet['!cols'] = columns.map(col => ({ wch: col.width || 12 }));
  }
  
  // Freeze header row
  if (freezeHeader) {
    worksheet['!freeze'] = { xSplit: 0, ySplit: 1, topLeftCell: 'A2' };
  }
  
  // Apply styles if supported (requires pro version for full styling)
  // Basic styling through cell type
  if (headers.length > 0) {
    // Set header row to bold (using cell type)
    const headerRange = XLSX.utils.decode_range(worksheet['!ref'] || 'A1:A1');
    for (let col = headerRange.s.c; col <= headerRange.e.c; col++) {
      const cellAddress = XLSX.utils.encode_cell({ r: 0, c: col });
      if (worksheet[cellAddress]) {
        worksheet[cellAddress].s = {
          ...worksheet[cellAddress].s,
          font: { bold: true },
          fill: { fgColor: { rgb: "FFE0E0E0" } },
        };
      }
    }
  }
  
  return worksheet;
}

/**
 * Download Excel file
 */
function downloadExcel(buffer: ArrayBuffer, filename: string): void {
  const blob = new Blob([buffer], {
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
  });
  
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  
  link.href = url;
  link.download = filename.endsWith('.xlsx') ? filename : `${filename}.xlsx`;
  link.style.display = 'none';
  
  document.body.appendChild(link);
  link.click();
  
  setTimeout(() => {
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }, 100);
}

/**
 * Export with cell types and formatting
 */
export function createExcelWithFormats<T>(
  data: T[],
  columns: ExcelExportOptions['columns']
): XLSX.WorkBook {
  const workbook = XLSX.utils.book_new();
  
  // Create worksheet with formatted data
  const wsData = [
    columns!.map(col => col.header), // Headers
    ...data.map(row => 
      columns!.map(col => {
        const value = (row as any)[col.key];
        
        // Apply formatter if exists
        if (col.formatter) {
          return col.formatter(value, row);
        }
        
        // Default formatting based on value type
        if (value instanceof Date) {
          return {
            v: value,
            t: 'd',
            z: col.format || 'yyyy-mm-dd',
          };
        }
        
        if (typeof value === 'number') {
          return {
            v: value,
            t: 'n',
            z: col.format || '#,##0.00',
          };
        }
        
        return value;
      })
    ),
  ];
  
  const worksheet = XLSX.utils.aoa_to_sheet(wsData);
  
  // Set column widths
  worksheet['!cols'] = columns!.map(col => ({ wch: col.width || 12 }));
  
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Data');
  
  return workbook;
}

// Example usage:
/*
const salesData = [
  { id: 1, date: new Date('2024-01-15'), product: 'Product A', quantity: 10, price: 29.99, total: 299.90 },
  { id: 2, date: new Date('2024-01-16'), product: 'Product B', quantity: 5, price: 49.99, total: 249.95 },
];

const salesColumns = [
  { key: 'id', header: 'ID', width: 8 },
  { key: 'date', header: 'Date', width: 12, format: 'yyyy-mm-dd' },
  { key: 'product', header: 'Product', width: 20 },
  { key: 'quantity', header: 'Quantity', width: 10 },
  { key: 'price', header: 'Price', width: 12, format: '$#,##0.00' },
  { key: 'total', header: 'Total', width: 12, format: '$#,##0.00' },
];

exportToExcel(salesData, {
  filename: 'sales-report',
  columns: salesColumns,
  freezeHeader: true,
  autoFitColumns: true,
});

// Multiple sheets example
exportToExcel([], {
  filename: 'multi-sheet-report',
  sheets: [
    {
      name: 'Sales',
      data: salesData,
      columns: salesColumns,
    },
    {
      name: 'Customers',
      data: customersData,
      columns: customerColumns,
    },
  ],
});
*/

§ 3.2 EXCELJS (ADVANCED FORMATTING)

typescript
// lib/export/excel-advanced.ts
import ExcelJS from 'exceljs';

export interface AdvancedExcelOptions {
  filename?: string;
  sheets: Array<{
    name: string;
    data: any[];
    columns: Array<{
      key: string;
      header: string;
      width?: number;
      style?: Partial<ExcelJS.Style>;
      numFmt?: string;
      alignment?: Partial<ExcelJS.Alignment>;
      border?: Partial<ExcelJS.Borders>;
      fill?: Partial<ExcelJS.Fill>;
    }>;
    options?: {
      freezeHeader?: boolean;
      autoFilter?: boolean;
      printArea?: string;
      headerRowStyle?: Partial<ExcelJS.Style>;
      alternateRowColors?: boolean;
    };
  }>;
  workbookOptions?: {
    creator?: string;
    lastModifiedBy?: string;
    created?: Date;
    modified?: Date;
    company?: string;
  };
}

/**
 * Export with advanced formatting using ExcelJS
 */
export async function exportToAdvancedExcel(
  options: AdvancedExcelOptions
): Promise<Blob> {
  const {
    filename = `export-${new Date().toISOString().slice(0, 10)}.xlsx`,
    sheets,
    workbookOptions,
  } = options;
  
  // Create workbook
  const workbook = new ExcelJS.Workbook();
  
  // Set workbook properties
  workbook.creator = workbookOptions?.creator || 'My App';
  workbook.lastModifiedBy = workbookOptions?.lastModifiedBy || 'My App';
  workbook.created = workbookOptions?.created || new Date();
  workbook.modified = workbookOptions?.modified || new Date();
  workbook.company = workbookOptions?.company || 'My Company';
  
  // Add sheets
  for (const sheetConfig of sheets) {
    const worksheet = workbook.addWorksheet(sheetConfig.name);
    
    // Add header row
    const headerRow = worksheet.addRow(sheetConfig.columns.map(col => col.header));
    
    // Apply header style
    headerRow.font = { 
      bold: true, 
      size: 11,
      color: { argb: 'FFFFFFFF' },
    };
    headerRow.fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: { argb: 'FF4472C4' }, // Blue background
    };
    headerRow.alignment = { 
      vertical: 'middle', 
      horizontal: 'center',
      wrapText: true,
    };
    headerRow.border = {
      top: { style: 'thin' },
      left: { style: 'thin' },
      bottom: { style: 'thin' },
      right: { style: 'thin' },
    };
    
    // Set column widths
    sheetConfig.columns.forEach((col, index) => {
      const column = worksheet.getColumn(index + 1);
      column.width = col.width || 15;
      
      // Set column style
      if (col.style) {
        column.style = col.style;
      }
      
      if (col.numFmt) {
        column.numFmt = col.numFmt;
      }
    });
    
    // Add data rows
    sheetConfig.data.forEach((rowData, rowIndex) => {
      const row = worksheet.addRow(
        sheetConfig.columns.map(col => rowData[col.key])
      );
      
      // Apply column-specific styling
      sheetConfig.columns.forEach((col, colIndex) => {
        const cell = row.getCell(colIndex + 1);
        
        if (col.alignment) {
          cell.alignment = col.alignment;
        }
        
        if (col.border) {
          cell.border = col.border;
        }
        
        if (col.fill) {
          cell.fill = col.fill;
        }
        
        // Alternate row colors
        if (sheetConfig.options?.alternateRowColors && rowIndex % 2 === 0) {
          cell.fill = {
            type: 'pattern',
            pattern: 'solid',
            fgColor: { argb: 'FFF2F2F2' }, // Light gray
          };
        }
        
        // Format numbers
        if (col.numFmt && typeof cell.value === 'number') {
          cell.numFmt = col.numFmt;
        }
      });
    });
    
    // Freeze header row
    if (sheetConfig.options?.freezeHeader !== false) {
      worksheet.views = [
        { state: 'frozen', xSplit: 0, ySplit: 1, activeCell: 'A2' },
      ];
    }
    
    // Add auto-filter
    if (sheetConfig.options?.autoFilter) {
      worksheet.autoFilter = {
        from: { row: 1, column: 1 },
        to: { row: 1, column: sheetConfig.columns.length },
      };
    }
    
    // Set print area
    if (sheetConfig.options?.printArea) {
      worksheet.pageSetup.printArea = sheetConfig.options.printArea;
    }
  }
  
  // Generate buffer
  const buffer = await workbook.xlsx.writeBuffer();
  
  // Create blob and trigger download
  const blob = new Blob([buffer], {
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
  });
  
  downloadExcelJS(blob, filename);
  
  return blob;
}

function downloadExcelJS(blob: Blob, filename: string): void {
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  
  link.href = url;
  link.download = filename;
  link.style.display = 'none';
  
  document.body.appendChild(link);
  link.click();
  
  setTimeout(() => {
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }, 100);
}

// Example column definitions:
export const financialReportColumns = [
  {
    key: 'date',
    header: 'Date',
    width: 12,
    numFmt: 'yyyy-mm-dd',
    alignment: { horizontal: 'center' },
  },
  {
    key: 'account',
    header: 'Account',
    width: 25,
    style: { font: { bold: true } },
  },
  {
    key: 'description',
    header: 'Description',
    width: 40,
  },
  {
    key: 'debit',
    header: 'Debit',
    width: 15,
    numFmt: '#,##0.00;[Red]-#,##0.00',
    alignment: { horizontal: 'right' },
    fill: { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FFF2F2F2' } },
  },
  {
    key: 'credit',
    header: 'Credit',
    width: 15,
    numFmt: '#,##0.00;[Red]-#,##0.00',
    alignment: { horizontal: 'right' },
    fill: { type: 'pattern', pattern: 'solid', fgColor: { argb: 'FFF2F2F2' } },
  },
  {
    key: 'balance',
    header: 'Balance',
    width: 15,
    numFmt: '#,##0.00;[Red]-#,##0.00',
    alignment: { horizontal: 'right' },
    style: { font: { bold: true, color: { argb: 'FF2E75B5' } } },
    border: {
      top: { style: 'thin', color: { argb: 'FF000000' } },
      bottom: { style: 'double', color: { argb: 'FF000000' } },
    },
  },
];

---

§ §4. PDF EXPORT

§ 4.1 JSPDF WITH AUTOTABLE

typescript
// lib/export/pdf-jspdf.ts
import jsPDF from 'jspdf';
import autoTable, { UserOptions } from 'jspdf-autotable';

export interface PDFExportOptions {
  /** Document title */
  title?: string;
  
  /** Filename */
  filename?: string;
  
  /** Column configuration */
  columns: Array<{
    /** Data key */
    key: string;
    /** Column header */
    header: string;
    /** Column width (auto if not specified) */
    width?: number;
    /** Cell alignment */
    align?: 'left' | 'center' | 'right';
    /** Value formatter */
    formatter?: (value: any, row: any) => string;
  }>;
  
  /** Data rows */
  data: any[];
  
  /** PDF options */
  pdf?: {
    orientation?: 'portrait' | 'landscape';
    unit?: 'mm' | 'cm' | 'in' | 'pt';
    format?: 'a4' | 'letter' | 'legal' | 'a3' | 'a5';
    margin?: {
      top?: number;
      right?: number;
      bottom?: number;
      left?: number;
    };
  };
  
  /** Table styling */
  styles?: {
    /** Header style */
    header?: Partial<{
      fillColor: [number, number, number] | string;
      textColor: [number, number, number] | string;
      fontStyle: 'normal' | 'bold' | 'italic' | 'bolditalic';
      fontSize: number;
    }>;
    /** Body style */
    body?: Partial<{
      fillColor: [number, number, number] | string;
      textColor: [number, number, number] | string;
      fontStyle: 'normal' | 'bold' | 'italic' | 'bolditalic';
      fontSize: number;
    }>;
    /** Alternating row colors */
    alternateRow?: boolean;
  };
  
  /** Footer content */
  footer?: {
    text?: string;
    pageNumber?: boolean;
    totalPages?: boolean;
  };
}

export interface PDFExportResult {
  /** PDF document */
  pdf: jsPDF;
  /** Number of pages */
  pageCount: number;
  /** Filename used */
  filename: string;
  /** Blob URL for download */
  blobUrl: string;
}

/**
 * Export data to PDF using jsPDF and autoTable
 */
export function exportToPDF(options: PDFExportOptions): PDFExportResult {
  const {
    title = 'Export',
    filename = `export-${new Date().toISOString().slice(0, 10)}.pdf`,
    columns,
    data,
    pdf: pdfOptions = {},
    styles = {},
    footer = {},
  } = options;
  
  // Default PDF options
  const {
    orientation = 'portrait',
    unit = 'mm',
    format = 'a4',
    margin = {},
  } = pdfOptions;
  
  const {
    top = 20,
    right = 10,
    bottom = 20,
    left = 10,
  } = margin;
  
  // Create PDF document
  const pdf = new jsPDF({
    orientation,
    unit,
    format,
    compress: true,
  });
  
  // Set document properties
  pdf.setProperties({
    title,
    subject: 'Export',
    creator: 'My Application',
    author: 'My Application',
  });
  
  // Add title
  if (title) {
    pdf.setFontSize(16);
    pdf.setFont('helvetica', 'bold');
    pdf.text(title, left, top);
  }
  
  // Prepare table data
  const tableColumns = columns.map(col => ({
    header: col.header,
    dataKey: col.key,
  }));
  
  const tableRows = data.map(row => {
    const tableRow: Record<string, any> = {};
    
    columns.forEach(col => {
      const value = row[col.key];
      tableRow[col.key] = col.formatter ? col.formatter(value, row) : value;
    });
    
    return tableRow;
  });
  
  // Configure table options
  const tableOptions: UserOptions = {
    startY: title ? top + 10 : top,
    margin: { top, right, bottom, left },
    head: [columns.map(col => col.header)],
    body: tableRows.map(row => columns.map(col => row[col.key])),
    headStyles: {
      fillColor: styles.header?.fillColor || [41, 128, 185],
      textColor: styles.header?.textColor || [255, 255, 255],
      fontStyle: styles.header?.fontStyle || 'bold',
      fontSize: styles.header?.fontSize || 10,
    },
    bodyStyles: {
      textColor: styles.body?.textColor || [50, 50, 50],
      fontSize: styles.body?.fontSize || 9,
    },
    alternateRowStyles: styles.alternateRow ? {
      fillColor: [245, 245, 245],
    } : undefined,
    columnStyles: columns.reduce((acc, col, index) => {
      if (col.width || col.align) {
        acc[index] = {
          cellWidth: col.width,
          halign: col.align,
        };
      }
      return acc;
    }, {} as Record<string, any>),
    didDrawPage: (data) => {
      // Add footer
      if (footer.text || footer.pageNumber || footer.totalPages) {
        const pageCount = (pdf as any).internal.getNumberOfPages();
        
        let footerText = footer.text || '';
        
        if (footer.pageNumber) {
          footerText += ` ${data.pageNumber}`;
        }
        
        if (footer.totalPages) {
          footerText += ` / ${pageCount}`;
        }
        
        if (footerText.trim()) {
          pdf.setFontSize(8);
          pdf.setTextColor(100);
          pdf.text(
            footerText.trim(),
            pdf.internal.pageSize.width / 2,
            pdf.internal.pageSize.height - 10,
            { align: 'center' }
          );
        }
      }
    },
  };
  
  // Add table to PDF
  autoTable(pdf, tableOptions);
  
  // Get final page count
  const pageCount = (pdf as any).internal.getNumberOfPages();
  
  // Trigger download
  const blobUrl = downloadPDF(pdf, filename);
  
  return {
    pdf,
    pageCount,
    filename,
    blobUrl,
  };
}

/**
 * Download PDF
 */
function downloadPDF(pdf: jsPDF, filename: string): string {
  const blob = pdf.output('blob');
  const url = URL.createObjectURL(blob);
  
  const link = document.createElement('a');
  link.href = url;
  link.download = filename;
  link.style.display = 'none';
  
  document.body.appendChild(link);
  link.click();
  
  setTimeout(() => {
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }, 100);
  
  return url;
}

/**
 * Export invoice to PDF
 */
export function exportInvoiceToPDF(invoiceData: any): PDFExportResult {
  const { customer, items, totals } = invoiceData;
  
  const pdf = new jsPDF();
  
  // Add company header
  pdf.setFontSize(20);
  pdf.setFont('helvetica', 'bold');
  pdf.text('INVOICE', 105, 20, { align: 'center' });
  
  pdf.setFontSize(10);
  pdf.setFont('helvetica', 'normal');
  pdf.text('My Company LLC', 105, 30, { align: 'center' });
  pdf.text('123 Business St, City, Country', 105, 35, { align: 'center' });
  pdf.text('contact@mycompany.com | +1 234 567 890', 105, 40, { align: 'center' });
  
  // Customer info
  pdf.setFont('helvetica', 'bold');
  pdf.text('Bill To:', 20, 60);
  pdf.setFont('helvetica', 'normal');
  pdf.text(customer.name, 20, 67);
  pdf.text(customer.address, 20, 72);
  pdf.text(customer.email, 20, 77);
  
  // Invoice details
  pdf.setFont('helvetica', 'bold');
  pdf.text('Invoice #:', 120, 60);
  pdf.text('Date:', 120, 67);
  pdf.text('Due Date:', 120, 74);
  
  pdf.setFont('helvetica', 'normal');
  pdf.text(invoiceData.invoiceNumber, 160, 60);
  pdf.text(invoiceData.date, 160, 67);
  pdf.text(invoiceData.dueDate, 160, 74);
  
  // Items table
  autoTable(pdf, {
    startY: 90,
    head: [['Description', 'Quantity', 'Unit Price', 'Total']],
    body: items.map((item: any) => [
      item.description,
      item.quantity,
      `$${item.unitPrice.toFixed(2)}`,
      `$${item.total.toFixed(2)}`,
    ]),
    headStyles: {
      fillColor: [41, 128, 185],
      textColor: [255, 255, 255],
      fontStyle: 'bold',
    },
    columnStyles: {
      0: { cellWidth: 80 },
      1: { halign: 'center' },
      2: { halign: 'right' },
      3: { halign: 'right' },
    },
  });
  
  // Totals
  const finalY = (pdf as any).lastAutoTable.finalY + 10;
  
  pdf.setFont('helvetica', 'bold');
  pdf.text('Subtotal:', 120, finalY);
  pdf.text('Tax:', 120, finalY + 6);
  pdf.text('Total:', 120, finalY + 12);
  
  pdf.setFont('helvetica', 'normal');
  pdf.text(`$${totals.subtotal.toFixed(2)}`, 160, finalY, { align: 'right' });
  pdf.text(`$${totals.tax.toFixed(2)}`, 160, finalY + 6, { align: 'right' });
  pdf.text(`$${totals.total.toFixed(2)}`, 160, finalY + 12, { align: 'right' });
  
  // Terms
  pdf.setFontSize(8);
  pdf.setTextColor(100);
  pdf.text('Payment Terms: Net 30 days', 20, finalY + 30);
  pdf.text('Thank you for your business!', 105, finalY + 30, { align: 'center' });
  
  // Download
  const filename = `invoice-${invoiceData.invoiceNumber}.pdf`;
  const blobUrl = downloadPDF(pdf, filename);
  
  return {
    pdf,
    pageCount: 1,
    filename,
    blobUrl,
  };
}

---

**NOTA:** Questo è solo il §1-§4 (Formats, CSV, Excel, PDF Export) della documentazione completa. Continuerei con:

- §5: Data Import (CSV/Excel import, validation, mapping)
- §6: Bulk Operations (batch insert, upsert patterns)
- §7: Export UI Components
- §8: Background Export for Large Datasets
- §9: Complete Checklist

Il codice include:
- Export CSV con PapaParse (streaming per grandi dataset)
- Export Excel con SheetJS e ExcelJS (formattazione avanzata)
- Export PDF con jsPDF e autoTable
- Type safety completa
- Error handling
- Esempi pratici per ogni use case