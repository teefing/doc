---
  draft: true
  category: 专业知识
  tags:
    - TypeScript
  date: 2020-07-01
  title: 啃一段ts代码
  vssue-title: 啃一段ts代码
---
```ts
const tuple = <T extends string[]>(...args: T) => args;

const ButtonTypes = tuple('default', 'primary', 'ghost', 'dashed', 'link', 'text');

type ButtonType = typeof ButtonTypes[number]
```