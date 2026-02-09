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