# 20250929-【TS】使用const断言和keyof获取key值联合类型

```ts
export const fileTypes = {
  xlsx: 'application/vnd.ms-excel;charset=UTF-8',
  xls: 'application/vnd.ms-excel;charset=UTF-8',
  doc: 'application/msword',
  docx: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  zip: 'application/zip',
  pdf: 'application/pdf;chartset=UTF-8',
  png: 'image/png',
  jpg: 'image/jpeg',
  gif: 'image/gif',
  txt: 'text/plain'
} as const;

export type FileType = keyof typeof fileTypes;
```

![image-20250929180410560](https://s2.loli.net/2025/09/29/gzH1V3kCwaXs2mv.png)