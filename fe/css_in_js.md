### CSS-in-JS

安装：
```bash
yarn add @emotion/react @emotion/styled
```

使用：

```ts
import { useState, useEffect } from 'react';
import styled from '@emotion/styled';
// 可以书写行内css。支持变量
import { jsx } from '@emotion/react';
import List from './list';

// 给组件添加样式：需要组件支持添加样式
const RedFont = styled(List)`
color: red;
`;

const variable = 1.4;

const DefaultFontSize = styled.div`
font-size: ${variable}rem;
`;

// 支持变量
const CustomRow = styled.div<{
  gap?: number;
}>`
display: flex;
align-items: center;
> div {
  margin-right: ${props => typeof props.gap === 'number' ? props.gap + 'rem' : undefined}
}
`;
```
### Grid布局

父元素：

```css
.father {
    display: grid;
    // 每一行的设置，共3行
    grid-template-rows: 6rem 1fr 6rem;
    // 每一列的设置，共3列
    grid-template-columns: 20rem 1fr 20rem;
    // 设置网格：3*3网格的内容
    grid-template-areas: 
    "header header header"
    "nav main aside"
    "footer footer footer"
    ;
    // 高
    height: 100vh;
    // 每个模块之间的距离
    // grid-gap: 1rem;
}
```

子元素：

```css
// 给每一个子元素起个网格别名
.header {
    grid-area: header;
}

.nav {
    grid-area: nav;
}

.main {
    grid-area: main;
}

.aside {
    grid-area: aside;
}

.footer {
    grid-area: footer;
}
```

### Flex布局

做一个导航栏，左边有logo和按钮，右边是登出。

父元素：

```css
.father {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
}
```

左边：

```css
.left {
    display: flex;
    align-items: center;
}
```

总结：
1. 一维布局用Flex，二维布局有Grid
2. 内容数量不固定时，用Flex

