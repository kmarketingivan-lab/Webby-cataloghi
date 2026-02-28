# CATALOGO-EXPORT-IMPORT

CATALOGO-EXPORT-IMPORT per Next.js 14 + TypeScript
§ CSV EXPORT
Papaparse Usage
typescript
import Papa from 'papaparse';

export interface CsvExportOptions<T = any> {
  data: T[];
  columns: CsvColumn<T>[];
  filename?: string;
  delimiter?: string;
  bom?: boolean;
  skipEmptyLines?: boolean;
}

export interface CsvColumn<T> {
  key: keyof T | string;
  label: string;
  formatter?: (value: any, row: T) => string;
}

export async function exportToCsv<T>(options: CsvExportOptions<T>): Promise<void> {
  const {
    data,
    columns,
    filename = 'export.csv',
    delimiter = ',',
    bom = true,
    skipEmptyLines = true
  } = options;

  // Transform data according to column definitions
  const transformedData = data.map(row => {
    const transformedRow: Record<string, string> = {};
    columns.forEach(column => {
      let value = column.key ? row[column.key as keyof T] : '';
      if (column.formatter) {
        value = column.formatter(value, row);
      }
      transformedRow[column.label] = String(value ?? '');
    });
    return transformedRow;
  });

  // Configure PapaParse
  const csv = Papa.unparse(transformedData, {
    delimiter,
    skipEmptyLines,
    header: true,
    quotes: true,
    escapeFormulae: true // Prevent CSV injection
  });

  // Add UTF-8 BOM for Excel compatibility
  const bomPrefix = bom ? '\uFEFF' : '';
  const csvContent = bomPrefix + csv;

  // Create and trigger download
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
  const link = document.createElement('a');
  const url = URL.createObjectURL(blob);
  
  link.setAttribute('href', url);
  link.setAttribute('download', filename);
  link.style.visibility = 'hidden';
  
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  
  URL.revokeObjectURL(url);
}
Large Dataset Streaming
typescript
import { Transform, Readable } from 'stream';
import { stringify } from 'csv-stringify';

export class CsvStreamTransformer<T> extends Transform {
  private columns: CsvColumn<T>[];
  private isFirstChunk = true;

  constructor(columns: CsvColumn<T>[]) {
    super({ objectMode: true });
    this.columns = columns;
  }

  _transform(chunk: T, encoding: string, callback: Function) {
    try {
      // Add headers for first chunk
      if (this.isFirstChunk) {
        const headers = this.columns.map(col => col.label);
        this.push(headers.join(',') + '\n');
        this.isFirstChunk = false;
      }

      // Transform row
      const row = this.columns.map(column => {
        let value = column.key ? chunk[column.key as keyof T] : '';
        if (column.formatter) {
          value = column.formatter(value, chunk);
        }
        // Escape quotes and commas
        const escapedValue = String(value ?? '').replace(/"/g, '""');
        return `"${escapedValue}"`;
      });

      this.push(row.join(',') + '\n');
      callback();
    } catch (error) {
      callback(error);
    }
  }
}

export async function streamLargeCsv<T>(
  dataStream: Readable,
  columns: CsvColumn<T>[],
  res: NextApiResponse
): Promise<void> {
  res.setHeader('Content-Type', 'text/csv; charset=utf-8');
  res.setHeader('Content-Disposition', 'attachment; filename="large-export.csv"');
  
  // Add BOM for Excel
  res.write('\uFEFF');
  
  const transformer = new CsvStreamTransformer(columns);
  
  dataStream.pipe(transformer).pipe(res);
}
Custom Formatters
typescript
export const csvFormatters = {
  date: (value: Date | string): string => {
    if (!value) return '';
    const date = value instanceof Date ? value : new Date(value);
    return date.toISOString().split('T')[0]; // YYYY-MM-DD
  },
  
  currency: (value: number, currency = 'EUR'): string => {
    return new Intl.NumberFormat('it-IT', {
      style: 'currency',
      currency
    }).format(value || 0);
  },
  
  boolean: (value: boolean): string => {
    return value ? 'Sì' : 'No';
  },
  
  array: (value: any[], separator = '; '): string => {
    return value?.join(separator) || '';
  },
  
  json: (value: any): string => {
    return JSON.stringify(value);
  }
};

// Usage example
const columns: CsvColumn<User>[] = [
  {
    key: 'name',
    label: 'Nome',
    formatter: (value) => value.toUpperCase()
  },
  {
    key: 'createdAt',
    label: 'Data Creazione',
    formatter: csvFormatters.date
  },
  {
    key: 'salary',
    label: 'Stipendio',
    formatter: (value) => csvFormatters.currency(value)
  }
];
Header Mapping
typescript
export interface HeaderMapping {
  source: string;
  target: string;
  required?: boolean;
  validator?: (value: any) => boolean;
}

export function mapCsvHeaders(
  data: Record<string, any>[],
  mappings: HeaderMapping[]
): Record<string, any>[] {
  return data.map(row => {
    const mappedRow: Record<string, any> = {};
    
    mappings.forEach(mapping => {
      const value = row[mapping.source];
      
      // Validate if validator exists
      if (mapping.validator && !mapping.validator(value)) {
        throw new Error(`Invalid value for ${mapping.source}: ${value}`);
      }
      
      mappedRow[mapping.target] = value;
    });
    
    return mappedRow;
  });
}
UTF-8 BOM for Excel
typescript
export function addUtf8Bom(content: string): string {
  return '\uFEFF' + content;
}

export function generateExcelCompatibleCsv(
  data: any[],
  columns: CsvColumn<any>[]
): Blob {
  const csv = Papa.unparse(data, {
    delimiter: ';', // Excel Italian locale uses semicolon
    header: true,
    quotes: true
  });
  
  const csvWithBom = addUtf8Bom(csv);
  
  return new Blob([csvWithBom], {
    type: 'text/csv; charset=utf-8;'
  });
}

// Server-side Excel CSV generation
export async function generateExcelCsvResponse(
  data: any[],
  columns: CsvColumn<any>[],
  filename: string = 'export.csv'
): Promise<Response> {
  const csv = Papa.unparse(data, { delimiter: ';', header: true });
  const csvWithBom = addUtf8Bom(csv);
  
  return new Response(csvWithBom, {
    headers: {
      'Content-Type': 'text/csv; charset=utf-8',
      'Content-Disposition': `attachment; filename="${filename}"`,
      'Pragma': 'public',
      'Cache-Control': 'no-cache'
    }
  });
}
§ EXCEL EXPORT (XLSX)
SheetJS/xlsx Library
typescript
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';

export interface ExcelExportOptions<T = any> {
  data: T[] | Record<string, T[]>;
  filename?: string;
  sheetName?: string | string[];
  headers?: string[][];
  styling?: ExcelStylingOptions;
  formulas?: ExcelFormula[];
  images?: ExcelImage[];
}

export interface ExcelStylingOptions {
  headerStyle?: XLSX.CellStyle;
  dataStyle?: XLSX.CellStyle;
  columnWidths?: number[];
  freezePane?: { x: number; y: number };
}

export interface ExcelFormula {
  cell: string; // A1 notation
  formula: string;
  value?: any;
}

export interface ExcelImage {
  position: { row: number; col: number };
  base64Data: string;
  type: 'png' | 'jpeg';
  size: { width: number; height: number };
}

export function exportToExcel<T>(options: ExcelExportOptions<T>): void {
  const {
    data,
    filename = 'export.xlsx',
    sheetName = 'Sheet1',
    headers,
    styling,
    formulas,
    images
  } = options;

  const wb = XLSX.utils.book_new();
  
  // Handle multiple sheets or single sheet
  if (Array.isArray(sheetName) && !Array.isArray(data)) {
    throw new Error('Multiple sheet names provided but data is not a record');
  }
  
  if (typeof data === 'object' && !Array.isArray(data)) {
    // Multiple sheets
    Object.entries(data).forEach(([sheet, sheetData], index) => {
      const ws = createWorksheet(sheetData as T[], {
        sheetName: sheetName[index] || sheet,
        headers: headers?.[index],
        styling,
        formulas,
        images
      });
      XLSX.utils.book_append_sheet(wb, ws, sheetName[index] || sheet);
    });
  } else {
    // Single sheet
    const ws = createWorksheet(data as T[], {
      sheetName: sheetName as string,
      headers: Array.isArray(headers) ? headers[0] : headers,
      styling,
      formulas,
      images
    });
    XLSX.utils.book_append_sheet(wb, ws, sheetName as string);
  }
  
  // Generate and save file
  const wbout = XLSX.write(wb, {
    bookType: 'xlsx',
    type: 'array',
    bookSST: true,
    compression: true
  });
  
  const blob = new Blob([wbout], { type: 'application/octet-stream' });
  saveAs(blob, filename);
}

function createWorksheet<T>(
  data: T[],
  options: {
    sheetName: string;
    headers?: string[];
    styling?: ExcelStylingOptions;
    formulas?: ExcelFormula[];
    images?: ExcelImage[];
  }
): XLSX.WorkSheet {
  // Convert data to worksheet
  const ws = XLSX.utils.json_to_sheet(data, {
    header: options.headers
  });
  
  // Apply styling
  if (options.styling) {
    applyWorksheetStyling(ws, options.styling);
  }
  
  // Add formulas
  if (options.formulas) {
    options.formulas.forEach(formula => {
      ws[formula.cell] = { f: formula.formula, v: formula.value };
    });
  }
  
  return ws;
}

function applyWorksheetStyling(ws: XLSX.WorkSheet, styling: ExcelStylingOptions): void {
  const range = XLSX.utils.decode_range(ws['!ref'] || 'A1');
  
  // Apply header styling
  if (styling.headerStyle) {
    for (let C = range.s.c; C <= range.e.c; ++C) {
      const cell = XLSX.utils.encode_cell({ r: 0, c: C });
      if (ws[cell]) {
        ws[cell].s = styling.headerStyle;
      }
    }
  }
  
  // Apply column widths
  if (styling.columnWidths) {
    ws['!cols'] = styling.columnWidths.map(width => ({ wch: width }));
  }
  
  // Apply freeze pane
  if (styling.freezePane) {
    ws['!freeze'] = {
      xSplit: styling.freezePane.x,
      ySplit: styling.freezePane.y,
      topLeftCell: 'B2',
      activePane: 'bottomRight',
      state: 'frozen'
    };
  }
}
Multiple Sheets
typescript
export interface MultiSheetExcelExport {
  sheets: {
    name: string;
    data: any[];
    columns: ExcelColumn[];
    options?: SheetOptions;
  }[];
  filename: string;
  globalOptions?: GlobalExcelOptions;
}

export interface ExcelColumn {
  key: string;
  label: string;
  width?: number;
  style?: XLSX.CellStyle;
  format?: string;
}

export function exportMultiSheetExcel(options: MultiSheetExcelExport): void {
  const wb = XLSX.utils.book_new();
  
  options.sheets.forEach(sheetConfig => {
    // Prepare data with proper formatting
    const formattedData = sheetConfig.data.map(row => {
      const formattedRow: Record<string, any> = {};
      sheetConfig.columns.forEach(col => {
        let value = row[col.key];
        
        // Apply formatting if specified
        if (col.format) {
          if (col.format.startsWith('date')) {
            value = new Date(value).toLocaleDateString();
          } else if (col.format.startsWith('currency')) {
            value = new Intl.NumberFormat('it-IT', {
              style: 'currency',
              currency: 'EUR'
            }).format(value);
          }
        }
        
        formattedRow[col.label] = value;
      });
      return formattedRow;
    });
    
    // Create worksheet
    const ws = XLSX.utils.json_to_sheet(formattedData);
    
    // Apply column widths
    if (sheetConfig.columns.some(col => col.width)) {
      ws['!cols'] = sheetConfig.columns.map(col => ({
        wch: col.width || 20
      }));
    }
    
    // Apply styling
    if (sheetConfig.options?.styling) {
      applySheetStyling(ws, sheetConfig.options.styling);
    }
    
    XLSX.utils.book_append_sheet(wb, ws, sheetConfig.name);
  });
  
  // Write file
  const wbout = XLSX.write(wb, {
    bookType: 'xlsx',
    type: 'array'
  });
  
  saveAs(new Blob([wbout], { type: 'application/octet-stream' }), options.filename);
}
Styling (colors, fonts, borders)
typescript
export interface ExcelStyle {
  font?: {
    name?: string;
    sz?: number;
    color?: { rgb: string };
    bold?: boolean;
    italic?: boolean;
    underline?: boolean;
  };
  fill?: {
    patternType?: string;
    fgColor?: { rgb: string };
    bgColor?: { rgb: string };
  };
  border?: {
    top?: BorderStyle;
    bottom?: BorderStyle;
    left?: BorderStyle;
    right?: BorderStyle;
  };
  alignment?: {
    horizontal?: 'left' | 'center' | 'right';
    vertical?: 'top' | 'center' | 'bottom';
    wrapText?: boolean;
  };
  numberFormat?: string;
}

export interface BorderStyle {
  style: 'thin' | 'medium' | 'thick' | 'dashed' | 'dotted';
  color: { rgb: string };
}

export const excelStyles = {
  header: {
    font: { bold: true, color: { rgb: 'FFFFFF' }, sz: 12 },
    fill: { patternType: 'solid', fgColor: { rgb: '4472C4' } },
    alignment: { horizontal: 'center', vertical: 'center' },
    border: {
      bottom: { style: 'medium', color: { rgb: '000000' } }
    }
  } as ExcelStyle,
  
  currency: {
    numberFormat: '€#,##0.00'
  } as ExcelStyle,
  
  date: {
    numberFormat: 'dd/mm/yyyy'
  } as ExcelStyle,
  
  percentage: {
    numberFormat: '0.00%'
  } as ExcelStyle,
  
  warning: {
    fill: { patternType: 'solid', fgColor: { rgb: 'FFEB9C' } },
    font: { color: { rgb: '9C6500' } }
  } as ExcelStyle,
  
  success: {
    fill: { patternType: 'solid', fgColor: { rgb: 'C6EFCE' } },
    font: { color: { rgb: '006100' } }
  } as ExcelStyle
};

export function createStyledWorksheet(
  data: any[],
  columns: Array<{
    key: string;
    label: string;
    style?: ExcelStyle;
    width?: number;
  }>
): XLSX.WorkSheet {
  // Create worksheet from data
  const ws = XLSX.utils.json_to_sheet(data, { header: columns.map(c => c.key) });
  
  // Get range
  const range = XLSX.utils.decode_range(ws['!ref'] || 'A1');
  
  // Apply column styles
  columns.forEach((col, colIndex) => {
    // Apply header style
    const headerCell = XLSX.utils.encode_cell({ r: 0, c: colIndex });
    if (ws[headerCell]) {
      ws[headerCell].v = col.label;
      ws[headerCell].s = excelStyles.header;
    }
    
    // Apply data styles
    for (let rowIndex = 1; rowIndex <= range.e.r; rowIndex++) {
      const cell = XLSX.utils.encode_cell({ r: rowIndex, c: colIndex });
      if (ws[cell] && col.style) {
        ws[cell].s = col.style;
      }
    }
  });
  
  // Set column widths
  ws['!cols'] = columns.map(col => ({ wch: col.width || 20 }));
  
  return ws;
}
Formulas
typescript
export function addExcelFormulas(
  ws: XLSX.WorkSheet,
  formulas: Array<{
    cell: string; // Target cell (e.g., "D10")
    formula: string; // Excel formula (e.g., "SUM(D2:D9)")
    format?: string;
  }>
): void {
  formulas.forEach(formula => {
    const cellAddress = XLSX.utils.encode_cell(XLSX.utils.decode_cell(formula.cell));
    
    ws[cellAddress] = {
      f: formula.formula,
      t: 'n' // Number type
    };
    
    if (formula.format) {
      ws[cellAddress].z = formula.format;
    }
  });
}

// Example usage with common formulas
export const commonFormulas = {
  sum: (startCell: string, endCell: string, targetCell: string) => ({
    cell: targetCell,
    formula: `SUM(${startCell}:${endCell})`,
    format: '#,##0.00'
  }),
  
  average: (startCell: string, endCell: string, targetCell: string) => ({
    cell: targetCell,
    formula: `AVERAGE(${startCell}:${endCell})`,
    format: '0.00'
  }),
  
  percentage: (numeratorCell: string, denominatorCell: string, targetCell: string) => ({
    cell: targetCell,
    formula: `${numeratorCell}/${denominatorCell}`,
    format: '0.00%'
  }),
  
  vlookup: (lookupValue: string, tableRange: string, colIndex: number, targetCell: string) => ({
    cell: targetCell,
    formula: `VLOOKUP(${lookupValue},${tableRange},${colIndex},FALSE)`
  })
};

// Generate dynamic formulas based on data length
export function generateAutoFormulas(
  dataLength: number,
  columnIndex: number
): Array<{ cell: string; formula: string }> {
  const formulas = [];
  
  // Last row index (0-indexed, header is row 0)
  const lastDataRow = dataLength;
  
  // Add SUM formula at the bottom
  const sumCell = XLSX.utils.encode_cell({ r: lastDataRow + 1, c: columnIndex });
  const sumRangeStart = XLSX.utils.encode_cell({ r: 1, c: columnIndex });
  const sumRangeEnd = XLSX.utils.encode_cell({ r: lastDataRow, c: columnIndex });
  
  formulas.push({
    cell: sumCell,
    formula: `SUM(${sumRangeStart}:${sumRangeEnd})`
  });
  
  // Add AVERAGE formula
  const avgCell = XLSX.utils.encode_cell({ r: lastDataRow + 2, c: columnIndex });
  formulas.push({
    cell: avgCell,
    formula: `AVERAGE(${sumRangeStart}:${sumRangeEnd})`
  });
  
  return formulas;
}
Images in Excel
typescript
export interface ExcelImageInsert {
  worksheet: XLSX.WorkSheet;
  image: {
    base64: string;
    type: 'png' | 'jpeg' | 'gif';
    position: {
      cell: string; // Starting cell (e.g., "A1")
      offset?: { x: number; y: number }; // Offset in points
    };
    size?: {
      width?: number; // In points
      height?: number; // In points
      scale?: number; // Scale factor
    };
  };
}

export function addImageToWorksheet(options: ExcelImageInsert): XLSX.WorkSheet {
  const { worksheet, image } = options;
  
  // In a real implementation, you would need to use a library that supports images
  // SheetJS Community Edition doesn't support images directly
  // This is a conceptual implementation
  
  // For actual image support, consider:
  // 1. Using SheetJS Pro version
  // 2. Using exceljs library instead
  // 3. Generating a template with images and populating data
  
  console.warn('Image insertion requires SheetJS Pro or alternative library');
  
  return worksheet;
}

// Alternative with exceljs (if switching library)
import ExcelJS from 'exceljs';

export async function createExcelWithImages(
  data: any[],
  images: Array<{
    imageBuffer: Buffer;
    extension: 'png' | 'jpeg';
    position: { row: number; col: number };
  }>
): Promise<Buffer> {
  const workbook = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet('Sheet1');
  
  // Add data
  worksheet.addRows(data);
  
  // Add images
  for (const img of images) {
    const image = workbook.addImage({
      buffer: img.imageBuffer,
      extension: img.extension
    });
    
    worksheet.addImage(image, {
      tl: { col: img.position.col, row: img.position.row },
      ext: { width: 100, height: 100 }
    });
  }
  
  return await workbook.xlsx.writeBuffer();
}
Large File Handling
typescript
export class ExcelStreamWriter {
  private workbook: XLSX.WorkBook;
  private currentRow: number = 0;
  private chunkSize: number = 10000;
  private worksheet: XLSX.WorkSheet;
  
  constructor(sheetName: string = 'Data') {
    this.workbook = XLSX.utils.book_new();
    this.worksheet = XLSX.utils.aoa_to_sheet([]);
    XLSX.utils.book_append_sheet(this.workbook, this.worksheet, sheetName);
    this.currentRow = 0;
  }
  
  addHeaders(headers: string[]): void {
    headers.forEach((header, index) => {
      const cell = XLSX.utils.encode_cell({ r: this.currentRow, c: index });
      this.worksheet[cell] = { v: header, t: 's' };
    });
    this.currentRow++;
  }
  
  addChunk(data: any[][]): void {
    data.forEach(row => {
      row.forEach((value, colIndex) => {
        const cell = XLSX.utils.encode_cell({ r: this.currentRow, c: colIndex });
        this.worksheet[cell] = {
          v: value,
          t: typeof value === 'number' ? 'n' : 's'
        };
      });
      this.currentRow++;
      
      // Optimize memory by limiting stored cells
      if (this.currentRow % this.chunkSize === 0) {
        this.worksheet['!ref'] = XLSX.utils.encode_range({
          s: { r: 0, c: 0 },
          e: { r: this.currentRow, c: 0 }
        });
      }
    });
  }
  
  getWorkbook(): XLSX.WorkBook {
    // Set final range
    const maxCol = this.getMaxColumn();
    this.worksheet['!ref'] = XLSX.utils.encode_range({
      s: { r: 0, c: 0 },
      e: { r: this.currentRow - 1, c: maxCol }
    });
    
    return this.workbook;
  }
  
  private getMaxColumn(): number {
    let maxCol = 0;
    Object.keys(this.worksheet).forEach(key => {
      if (key[0] !== '!') {
        const col = XLSX.utils.decode_cell(key).c;
        maxCol = Math.max(maxCol, col);
      }
    });
    return maxCol;
  }
}

// Server-side streaming for very large files
export async function streamLargeExcel(
  dataStream: AsyncIterable<any[]>,
  res: NextApiResponse
): Promise<void> {
  const writer = new ExcelStreamWriter();
  
  // Write headers
  writer.addHeaders(['ID', 'Name', 'Value', 'Date']);
  
  // Stream data
  for await (const chunk of dataStream) {
    writer.addChunk(chunk);
  }
  
  // Get final workbook
  const wb = writer.getWorkbook();
  
  // Stream to response
  res.setHeader('Content-Type', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
  res.setHeader('Content-Disposition', 'attachment; filename="large-export.xlsx"');
  
  const wbout = XLSX.write(wb, {
    bookType: 'xlsx',
    type: 'buffer',
    compression: true
  });
  
  res.send(wbout);
}
§ PDF EXPORT
@react-pdf/renderer
typescript
import React from 'react';
import { 
  Document, 
  Page, 
  Text, 
  View, 
  StyleSheet, 
  Image,
  PDFViewer,
  PDFDownloadLink,
  Font,
  BlobProvider
} from '@react-pdf/renderer';
import { format } from 'date-fns';
import { it } from 'date-fns/locale';

// Register fonts
Font.register({
  family: 'Roboto',
  fonts: [
    { src: '/fonts/Roboto-Regular.ttf' },
    { src: '/fonts/Roboto-Bold.ttf', fontWeight: 'bold' },
    { src: '/fonts/Roboto-Italic.ttf', fontWeight: 'normal', fontStyle: 'italic' }
  ]
});

Font.register({
  family: 'Times-Roman',
  src: '/fonts/Times-Roman.ttf'
});

// Define styles
export const pdfStyles = StyleSheet.create({
  page: {
    flexDirection: 'column',
    backgroundColor: '#FFFFFF',
    padding: 30,
    fontFamily: 'Roboto'
  },
  header: {
    marginBottom: 20,
    borderBottom: '2pt solid #333',
    paddingBottom: 10
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5
  },
  subtitle: {
    fontSize: 12,
    color: '#666',
    marginBottom: 10
  },
  section: {
    marginVertical: 10,
    padding: 10,
    backgroundColor: '#f5f5f5'
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#444'
  },
  table: {
    display: 'flex',
    width: 'auto',
    borderStyle: 'solid',
    borderWidth: 1,
    borderColor: '#ccc',
    marginVertical: 10
  },
  tableRow: {
    flexDirection: 'row',
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
    minHeight: 30,
    alignItems: 'center'
  },
  tableHeader: {
    backgroundColor: '#f0f0f0',
    fontWeight: 'bold'
  },
  tableCell: {
    padding: 8,
    fontSize: 10,
    flex: 1,
    borderRightWidth: 1,
    borderRightColor: '#ccc'
  },
  footer: {
    position: 'absolute',
    bottom: 30,
    left: 30,
    right: 30,
    fontSize: 10,
    color: '#666',
    borderTop: '1pt solid #ccc',
    paddingTop: 10,
    textAlign: 'center'
  },
  pageNumber: {
    position: 'absolute',
    bottom: 30,
    right: 30,
    fontSize: 10,
    color: '#666'
  }
});

// Base PDF Document Component
interface PdfDocumentProps {
  title: string;
  subtitle?: string;
  data: any[];
  columns: PdfColumn[];
  footerText?: string;
  includePageNumbers?: boolean;
}

export interface PdfColumn {
  key: string;
  label: string;
  width?: string;
  render?: (value: any, row: any) => React.ReactNode;
  align?: 'left' | 'center' | 'right';
}

export const PdfDocument: React.FC<PdfDocumentProps> = ({
  title,
  subtitle,
  data,
  columns,
  footerText = 'Documento generato automaticamente',
  includePageNumbers = true
}) => (
  <Document>
    <Page size="A4" style={pdfStyles.page}>
      {/* Header */}
      <View style={pdfStyles.header}>
        <Text style={pdfStyles.title}>{title}</Text>
        {subtitle && <Text style={pdfStyles.subtitle}>{subtitle}</Text>}
        <Text style={pdfStyles.subtitle}>
          Data generazione: {format(new Date(), 'dd MMMM yyyy', { locale: it })}
        </Text>
      </View>
      
      {/* Table */}
      <View style={pdfStyles.table}>
        {/* Table Header */}
        <View style={[pdfStyles.tableRow, pdfStyles.tableHeader]}>
          {columns.map((column, index) => (
            <Text 
              key={index} 
              style={[
                pdfStyles.tableCell, 
                { flex: column.width || 1, textAlign: column.align || 'left' }
              ]}
            >
              {column.label}
            </Text>
          ))}
        </View>
        
        {/* Table Rows */}
        {data.map((row, rowIndex) => (
          <View key={rowIndex} style={pdfStyles.tableRow}>
            {columns.map((column, colIndex) => (
              <View 
                key={colIndex} 
                style={[
                  pdfStyles.tableCell, 
                  { flex: column.width || 1 }
                ]}
              >
                {column.render ? (
                  column.render(row[column.key], row)
                ) : (
                  <Text style={{ textAlign: column.align || 'left' }}>
                    {row[column.key] || ''}
                  </Text>
                )}
              </View>
            ))}
          </View>
        ))}
      </View>
      
      {/* Footer */}
      <View style={pdfStyles.footer}>
        <Text>{footerText}</Text>
      </View>
      
      {/* Page Number */}
      {includePageNumbers && (
        <Text 
          style={pdfStyles.pageNumber}
          render={({ pageNumber, totalPages }) => (
            `Pagina ${pageNumber} di ${totalPages}`
          )}
          fixed
        />
      )}
    </Page>
  </Document>
);

// PDF Download Component
export const PdfDownloadButton: React.FC<PdfDocumentProps & { filename?: string }> = (props) => (
  <PDFDownloadLink
    document={<PdfDocument {...props} />}
    fileName={props.filename || `export-${format(new Date(), 'yyyy-MM-dd')}.pdf`}
  >
    {({ loading }) => (
      <button disabled={loading}>
        {loading ? 'Generando PDF...' : 'Scarica PDF'}
      </button>
    )}
  </PDFDownloadLink>
);

// PDF Viewer Component
export const PdfViewer: React.FC<PdfDocumentProps> = (props) => (
  <PDFViewer width="100%" height="600px">
    <PdfDocument {...props} />
  </PDFViewer>
);

// Blob Provider for custom handling
export const PdfBlobProvider: React.FC<
  PdfDocumentProps & {
    children: (props: { url: string | null; loading: boolean; error: Error | null }) => React.ReactNode;
  }
> = ({ children, ...pdfProps }) => (
  <BlobProvider document={<PdfDocument {...pdfProps} />}>
    {children}
  </BlobProvider>
);
Document Templates
typescript
// Invoice Template
export const InvoiceTemplate: React.FC<{
  invoice: Invoice;
  company: CompanyInfo;
  client: ClientInfo;
}> = ({ invoice, company, client }) => (
  <Document>
    <Page size="A4" style={pdfStyles.page}>
      {/* Company Header */}
      <View style={{ flexDirection: 'row', justifyContent: 'space-between', marginBottom: 20 }}>
        <View>
          <Text style={{ fontSize: 20, fontWeight: 'bold' }}>{company.name}</Text>
          <Text style={{ fontSize: 10 }}>{company.address}</Text>
          <Text style={{ fontSize: 10 }}>P.IVA: {company.vatNumber}</Text>
        </View>
        <View>
          <Text style={{ fontSize: 16, fontWeight: 'bold' }}>FATTURA</Text>
          <Text style={{ fontSize: 12 }}>N° {invoice.number}</Text>
          <Text style={{ fontSize: 10 }}>
            Data: {format(new Date(invoice.date), 'dd/MM/yyyy')}
          </Text>
        </View>
      </View>
      
      {/* Client Info */}
      <View style={{ marginBottom: 20, padding: 10, backgroundColor: '#f9f9f9' }}>
        <Text style={{ fontSize: 12, fontWeight: 'bold', marginBottom: 5 }}>CLIENTE</Text>
        <Text style={{ fontSize: 10 }}>{client.name}</Text>
        <Text style={{ fontSize: 10 }}>{client.address}</Text>
        <Text style={{ fontSize: 10 }}>P.IVA: {client.vatNumber}</Text>
      </View>
      
      {/* Invoice Items */}
      <View style={pdfStyles.table}>
        <View style={[pdfStyles.tableRow, pdfStyles.tableHeader]}>
          <Text style={[pdfStyles.tableCell, { flex: 3 }]}>Descrizione</Text>
          <Text style={[pdfStyles.tableCell, { flex: 1 }]}>Quantità</Text>
          <Text style={[pdfStyles.tableCell, { flex: 1 }]}>Prezzo</Text>
          <Text style={[pdfStyles.tableCell, { flex: 1 }]}>Totale</Text>
        </View>
        
        {invoice.items.map((item, index) => (
          <View key={index} style={pdfStyles.tableRow}>
            <Text style={[pdfStyles.tableCell, { flex: 3 }]}>{item.description}</Text>
            <Text style={[pdfStyles.tableCell, { flex: 1 }]}>{item.quantity}</Text>
            <Text style={[pdfStyles.tableCell, { flex: 1 }]}>
              {formatCurrency(item.unitPrice)}
            </Text>
            <Text style={[pdfStyles.tableCell, { flex: 1 }]}>
              {formatCurrency(item.total)}
            </Text>
          </View>
        ))}
      </View>
      
      {/* Totals */}
      <View style={{ marginTop: 20, alignItems: 'flex-end' }}>
        <View style={{ width: 200 }}>
          <View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
            <Text>Subtotale:</Text>
            <Text>{formatCurrency(invoice.subtotal)}</Text>
          </View>
          <View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
            <Text>IVA ({invoice.vatRate}%):</Text>
            <Text>{formatCurrency(invoice.vatAmount)}</Text>
          </View>
          <

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-03-EXPORT-IMPORT
Prompt ID: 3 / 19
Parte: 2
Exported: 2026-02-06T16:38:08.743Z
Characters: 2283
════════════════════════════════════════════════════════════

Readable,
  res: NextApiResponse,
  options: {
    filename?: string;
    prettyPrint?: boolean;
    isArray?: boolean;
  } = {}
): Promise<void> {
  const {
    filename = 'large-export.json',
    prettyPrint = true,
    isArray = true
  } = options;
  
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
  
  if (!prettyPrint) {
    // Simple streaming for minified JSON
    res.write(isArray ? '[' : '{');
    
    let first = true;
    dataStream.on('data', (chunk) => {
      if (!first) res.write(',');
      res.write(JSON.stringify(chunk));
      first = false;
    });
    
    dataStream.on('end', () => {
      res.write(isArray ? ']' : '}');
      res.end();
    });
    
    dataStream.on('error', (error) => {
      console.error('Stream error:', error);
      res.status(500).end();
    });
  } else {
    // Pretty printed streaming
    const transformer = new JsonStreamTransformer(isArray);
    dataStream.pipe(transformer).pipe(res);
  }
}

// Chunked JSON export for very large datasets
export class JsonChunkedExporter {
  private chunkSize: number;
  private currentChunk: any[] = [];
  private chunkIndex = 0;
  private totalExported = 0;
  
  constructor(chunkSize: number = 10000) {
    this.chunkSize = chunkSize;
  }
  
  async export(
    dataGenerator: AsyncGenerator<any> | any[],
    onChunk: (chunk: any[], index: number) => Promise<void>
  ): Promise<{ totalChunks: number; totalItems: number }> {
    let totalChunks = 0;
    
    if (Array.isArray(dataGenerator)) {
      // Handle array
      for (let i = 0; i < dataGenerator.length; i++) {
        this.currentChunk.push(dataGenerator[i]);
        this.totalExported++;
        
        if (this.currentChunk.length >= this.chunkSize) {
          await onChunk(this.currentChunk, this.chunkIndex);
          this.currentChunk = [];
          this.chunkIndex++;
          totalChunks++;
        }
      }
    } else {
      // Handle async generator
      for await (const item of dataGenerator) {
        this.currentChunk.push(item);
        this.totalExported++;
        
        if (this.currentChunk.length >= this.chunkSize) {
          await onChunk(this.currentChunk, this.chunkIndex);
          this.current

## § ADVANCED PATTERNS: EXPORT IMPORT

### Server Actions con Validazione

```typescript
// app/actions/export-import.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### EXPORT IMPORT - Utility Helper #524

```typescript
// lib/utils/export-import-helper-524.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #697

```typescript
// lib/utils/export-import-helper-697.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #882

```typescript
// lib/utils/export-import-helper-882.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #239

```typescript
// lib/utils/export-import-helper-239.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #79

```typescript
// lib/utils/export-import-helper-79.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #931

```typescript
// lib/utils/export-import-helper-931.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #907

```typescript
// lib/utils/export-import-helper-907.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #406

```typescript
// lib/utils/export-import-helper-406.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #353

```typescript
// lib/utils/export-import-helper-353.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #435

```typescript
// lib/utils/export-import-helper-435.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #816

```typescript
// lib/utils/export-import-helper-816.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #883

```typescript
// lib/utils/export-import-helper-883.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #741

```typescript
// lib/utils/export-import-helper-741.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #51

```typescript
// lib/utils/export-import-helper-51.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #519

```typescript
// lib/utils/export-import-helper-519.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #49

```typescript
// lib/utils/export-import-helper-49.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### EXPORT IMPORT - Utility Helper #599

```typescript
// lib/utils/export-import-helper-599.ts
import { z } from "zod";

interface EXPORTIMPORTConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class EXPORTIMPORTProcessor<TInput, TOutput> {
  private config: EXPORTIMPORTConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<EXPORTIMPORTConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<EXPORTIMPORTConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
