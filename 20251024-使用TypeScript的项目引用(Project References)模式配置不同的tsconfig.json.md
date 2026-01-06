# 20251024-使用TypeScript的项目引用(Project References)模式配置不同的tsconfig.json

这是 TypeScript 的 **项目引用（Project References）** 模式，将配置分成 3 个文件有以下原因：

## 📁 三个文件的职责划分

### 1️⃣ **tsconfig.json（主配置 - 协调器）**

```json
{
  "files": [],
  "compilerOptions": {
    "strict": true,
    "forceConsistentCasingInFileNames": true
  },
  "references": [
    {
      "path": "./tsconfig.node.json"
    },
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.vitest.json"
    }
  ]
}
```

**作用**：
- 🎯 **不直接编译任何文件**（`files: []`）
- 🎯 **引用其他子配置**（通过 `references`）
- 🎯 **定义全局共享规则**（如 `strict`、`forceConsistentCasingInFileNames`）

---

### 2️⃣ **tsconfig.app.json（应用代码配置）**

```json
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",
  "include": ["env.d.ts", "src/**/*", "src/**/*.vue"],
  "exclude": ["src/**/__tests__/*"],
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "verbatimModuleSyntax": false,

    "paths": {
      "@/*": ["./src/*"]
    },
    // 自定义配置
    "noImplicitAny": false, // 禁用 隐式的any错误
    "removeComments": true, // 移除 ts注释
    "experimentalDecorators": true, // 启用实验性的装饰器支持
    "strictFunctionTypes": false, // 禁用严格函数类型检查
    "strictNullChecks": false, // 禁用严格的空值检查
    "allowSyntheticDefaultImports": true, // 允许默认导入
    "forceConsistentCasingInFileNames": true // 强制在文件名中使
  }
}
```

**作用**：
- 🎯 **只处理 `src/` 目录**的业务代码
- 🎯 **使用浏览器环境**（DOM API）
- 🎯 **配置路径别名** `@/*`
- 🎯 **包含 Vue 文件**处理

---

### 3️⃣ **tsconfig.node.json（Node.js 工具配置）**

```json
{
  "extends": "@tsconfig/node22/tsconfig.json",
  "include": [
    "vite.config.*",
    "vitest.config.*",
    "cypress.config.*",
    "nightwatch.conf.*",
    "playwright.config.*",
    "eslint.config.*"
  ],
  "compilerOptions": {
    "noEmit": true,
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.node.tsbuildinfo",

    "module": "ESNext",
    "moduleResolution": "Bundler",
    "types": ["node"]
  }
}
```

**作用**：
- 🎯 **只处理配置文件**（vite、vitest、eslint 等）
- 🎯 **使用 Node.js 环境**（Node API）
- 🎯 **不生成编译产物**（`noEmit: true`）

---

## 🎯 为什么要这样分离？

### **1. 环境隔离**
```
浏览器环境 (DOM)          vs          Node.js 环境
     ↓                                      ↓
tsconfig.app.json                   tsconfig.node.json
  - window ✅                          - process ✅
  - document ✅                        - __dirname ✅
  - fetch ✅                           - require ✅
  - process ❌                         - window ❌
```

### **2. 编译优化**
```typescript
// 只修改了 vite.config.ts
// ✅ 只重新检查 tsconfig.node.json 的文件
// ❌ 不会重新检查整个 src/ 目录（节省时间）
```

### **3. 配置专一性**

| 配置       | 模块系统  | 类型定义   | 用途                     |
| ---------- | --------- | ---------- | ------------------------ |
| **app**    | ES Module | DOM        | 业务代码，运行在浏览器   |
| **node**   | ES Module | Node.js    | 工具配置，运行在 Node.js |
| **vitest** | ES Module | DOM + Node | 测试代码，需要两种环境   |

### **4. 避免类型冲突**

```typescript
// ❌ 如果不分离，可能出现：
// vite.config.ts 中想用 Node.js 的 process
// 但 DOM 类型会覆盖它，导致类型错误

// ✅ 分离后：
// tsconfig.node.json 明确只包含 Node 类型
// tsconfig.app.json 明确只包含 DOM 类型
```

---

## 🔄 工作流程

```
运行 tsc 或 IDE 检查
         ↓
  读取 tsconfig.json
         ↓
   发现 3 个引用
         ↓
    ┌────┴────┬────────┐
    ↓         ↓        ↓
  app.json  node.json  vitest.json
    ↓         ↓        ↓
  并行检查   并行检查   并行检查
    ↓         ↓        ↓
  src/**/*  *.config.*  **/__tests__/*
```

---

## 💡 好处总结

| 优势               | 说明                                |
| ------------------ | ----------------------------------- |
| ⚡ **编译速度更快** | 修改一个部分只重新检查相关文件      |
| 🎯 **类型更准确**   | 不同环境使用对应的类型定义          |
| 🔧 **配置更清晰**   | 每个配置文件职责单一                |
| 🚀 **增量编译**     | 通过 `tsBuildInfoFile` 缓存编译结果 |
| 🛡️ **避免冲突**     | Node.js API 和 DOM API 不会互相干扰 |

---

## 🤔 可以合并成一个吗？

可以，但会有问题：

```json
// ❌ 合并后的问题
{
  "include": ["src/**/*", "*.config.ts"],
  "compilerOptions": {
    "types": ["node", "dom"] // ⚠️ 类型定义冲突！
  }
}
```

- ❌ vite.config.ts 会引入不需要的 DOM 类型
- ❌ src/ 代码会引入不需要的 Node 类型
- ❌ 编译缓存失效，每次都要全量检查
- ❌ 配置混乱，难以维护

**结论**：分离配置是现代 TypeScript 项目的最佳实践，特别是在使用 Vite、Vue 等现代工具链时。