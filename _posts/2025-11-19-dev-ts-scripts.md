---
title: 使用 TypeScript 脚本优化小程序页面结构
author: 雷子
date: 2025-11-19 17:10:00 +0800
categories: [技术, TypeScript]
tags: [TypeScript, 脚本]
---

## 项目背景
Taro + TypeScript 构建微信小程序。

## 项目结构
小程序项目中，默认有
- `index.config.ts`
- `index.tsx`
- `style.less`

其中`index.config.ts` 用于配置小程序页面的窗口表现和功能，`index.tsx` 是页面的主要逻辑和结构，`style.less` 则用于定义页面的样式。

## 问题背景
在实际的项目开发中，我发现`index.tsx`中有太多的逻辑代码，导致文件变得臃肿且难以维护。为了提高代码的可读性和可维护性，我决定将一些通用的功能和逻辑抽离出来，放到单独的脚本文件中。 这样做不仅可以让`index.tsx`文件更加简洁，还能促进代码的复用。

## 实现步骤
我尝试创建了一个`logic.ts`文件，用于存放一些通用的逻辑函数和处理流程。然后在`index.tsx`中通过import语句引入这个logic.ts文件，并调用其中的函数来实现页面的功能。 这种方式让我能够更好地组织代码，使得每个文件都有明确的职责。同时，当需要修改某个逻辑时，只需在logic.ts中进行更改，而不必担心影响到index.tsx的其他部分。

```typescript
// logic.ts
export function useIndexLogic() {
  // 数据获取逻辑
    function fetchData() {
        // 模拟数据获取
        console.log("Fetching data...")
    }

    function onUserAction() {
        // 处理用户操作
        console.log("User action handled.")
    }

    return {
        fetchData,
        onUserAction,
    }
}

// index.tsx
import logic from './logic'

export default function Index() {
    const logic = useIndexLogic()

  useEffect(() => {
    logic.fetchData();
  }, [])

  return (
    <View>
      <Button onClick={logic.onUserAction}>点击我</Button>
    </View>
  )
}
```
这么做的好处是显而易见的。首先，`index.tsx`文件变得更加简洁，主要关注页面的结构和渲染逻辑，而将复杂的业务逻辑放在了`logic.ts`中。其次，这种模块化的方式使得代码更易于测试和维护。每个逻辑函数都可以独立测试，确保其功能正确。

## 利用脚本去优化页面结构
小程序在创建页面时，通常会生成多个文件，包括配置文件（如`index.config.ts`）、逻辑文件（如`index.tsx`）和样式文件（如`style.less`）。为了优化页面结构，可以编写一个TypeScript脚本来自动化这个过程。

### 创建脚本
#### 步骤 1.
创建scripts目录，并在其中创建一个名为`createPage.ts`的TypeScript脚本文件。

#### 步骤 2.
在`createPage.ts`中，编写代码以生成所需的文件结构和内容。例如：

```typescript
import * as fs from 'fs'
import * as path from 'path'

// 获取页面名称参数
const pageName = process.argv[2]

if (!pageName) {
  console.error('❌ 请提供页面名称!')
  console.log('使用方法: npm run create:page <pageName>')
  console.log('示例: npm run create:page mine')
  process.exit(1)
}

// 定义路径
const pagesDir = path.resolve(__dirname, '../src/pages')
const pageDir = path.join(pagesDir, pageName)

// 检查页面是否已存在
if (fs.existsSync(pageDir)) {
  console.error(`❌ 页面 "${pageName}" 已存在!`)
  process.exit(1)
}

// 创建页面目录
fs.mkdirSync(pageDir, { recursive: true })

// 页面名称首字母大写
const PageName = pageName.charAt(0).toUpperCase() + pageName.slice(1)

// ============ 模板内容 ============

// index.tsx 模板
const indexTsxContent = `import { View } from '@tarojs/components'
import LucaColumn from 'src/components/globals/column'
import LucaText from 'src/components/globals/text'
import use${PageName}Logic from './logic'

export default function ${PageName}Page() {
  const logic = use${PageName}Logic()

  return (
    <View>
      <LucaColumn style={{ padding: '20px' }}>
        <LucaText size="20px" weight="bold">
          ${PageName} 页面
        </LucaText>
        <LucaText color="#666">
          这是 ${pageName} 页面的内容
        </LucaText>
      </LucaColumn>
    </View>
  )
}
`

// index.config.ts 模板
const indexConfigContent = `export default definePageConfig({
  navigationBarTitleText: '${PageName}'
})
`

// logic.ts 模板
const logicContent = `import { useState } from 'react'
import { useNavigateRouter } from 'src/routers/navigate'

// 页面路由常量
export const ${pageName}RouteName = '/pages/${pageName}/index'

export default function use${PageName}Logic() {
  const router = useNavigateRouter()

  // 示例状态
  const [loading, setLoading] = useState(false)

  // 示例方法
  const handleClick = () => {
    console.log('${PageName} page clicked')
  }

  return {
    loading,
    handleClick,
  }
}
`

// index.less 模板 (可选)
// const indexLessContent = `.${pageName}-page {
//   padding: 20px;

//   &__title {
//     font-size: 20px;
//     font-weight: bold;
//   }

//   &__content {
//     margin-top: 16px;
//     color: #666;
//   }
// }
// `

// ============ 写入文件 ============

try {
  // 创建 index.tsx
  fs.writeFileSync(path.join(pageDir, 'index.tsx'), indexTsxContent)
  console.log(`✅ 创建 ${pageName}/index.tsx`)

  // 创建 index.config.ts
  fs.writeFileSync(path.join(pageDir, 'index.config.ts'), indexConfigContent)
  console.log(`✅ 创建 ${pageName}/index.config.ts`)

  // 创建 logic.ts
  fs.writeFileSync(path.join(pageDir, 'logic.ts'), logicContent)
  console.log(`✅ 创建 ${pageName}/logic.ts`)

  // 创建 index.less (可选)
  //   fs.writeFileSync(path.join(pageDir, 'index.less'), indexLessContent)
  //   console.log(`✅ 创建 ${pageName}/index.less`)

  // 自动添加路由到 app.config.ts
  const appConfigPath = path.resolve(__dirname, '../src/app.config.ts')
  if (fs.existsSync(appConfigPath)) {
    let appConfig = fs.readFileSync(appConfigPath, 'utf-8')
    const pageRoute = `pages/${pageName}/index`

    // 检查路由是否已存在
    if (!appConfig.includes(pageRoute)) {
      // 查找 pages 数组的结束位置 ]
      const pagesMatch = appConfig.match(/pages:\s*\[([\s\S]*?)\]/)
      if (pagesMatch) {
        const pagesContent = pagesMatch[1]
        // 在数组末尾添加新路由
        const newPagesContent = pagesContent.trim()
          ? `${pagesContent.trimEnd()},\n    '${pageRoute}'`
          : `\n    '${pageRoute}'\n  `

        appConfig = appConfig.replace(/pages:\s*\[([\s\S]*?)\]/, `pages: [${newPagesContent}]`)

        fs.writeFileSync(appConfigPath, appConfig)
        console.log(`✅ 自动添加路由到 app.config.ts`)
      } else {
        console.log(`⚠️  未找到 pages 数组,请手动添加路由: 'pages/${pageName}/index'`)
      }
    } else {
      console.log(`ℹ️  路由已存在于 app.config.ts`)
    }
  }

  console.log('\n✅ 页面创建成功!')
  console.log(`\n📂 页面路径: src/pages/${pageName}/`)
  console.log(`🔗 路由地址: pages/${pageName}/index`)
} catch (error) {
  console.error('❌ 创建页面失败:', error)
  process.exit(1)
}

```

### 运行脚本
在项目的`package.json`中添加一个脚本命令，以便运行这个TypeScript脚本：
```json
"scripts": {
  "create:page": "ts-node ./scripts/createPage.ts xxx"
}
```

### 脚本说明
在这个脚本中，我根据业务需求，定义了页面名称，并创建了对应的目录和文件。每个文件都包含了基本的模板内容，可以根据需要进行修改和扩展。 运行这个脚本后，会在指定的目录下生成一个新的页面结构，包括`index.tsx`、`index.config.ts`和`logic.ts`文件。这样可以大大简化页面创建的过程，提高开发效率。

在生成文件的基础上，我还添加了一个功能：自动将新页面的路由添加到`app.config.ts`文件中。 这个功能通过读取`app.config.ts`文件，检查是否已经存在该页面的路由，如果不存在，则将其添加到`pages`数组中。 这样做可以确保每次创建新页面时，路由配置都是最新的，避免了手动添加路由的繁琐步骤。


### 总结
通过编写TypeScript脚本来自动化小程序页面的创建和路由配置，不仅提高了开发效率，还确保了代码结构的清晰和一致性。这种方法对于大型项目尤为有用，因为它减少了人为错误的可能性，并促进了代码的模块化和可维护性。