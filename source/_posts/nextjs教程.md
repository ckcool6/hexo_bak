---
title: nextjs教程（一）
date: 2024-04-04 01:07:06
tags:
---

## 安装nextjs：
nextjs是一个react框架。

> npx create-next-app@latest

这是官方文档推荐的安装方式，
这里我用另一个命令安装别人已经写好的半成品：

> npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"

安装好之后，cd到`nextjs-dashboard`目录，运行`npm install`安装所需依赖，然后运行`npm run dev`查看效果。

## 样式

这个模板工程默认没有导入css，到`app`目录下找到`layout.tsx`文件，输入
```javascript
import '@/app/ui/global.css';
```
导入样式，使之生效。

## tailwind css

此工程默认采用tailwind css这个库来写样式，为什么不用原生写呢？当然原生也可以，react开发组件把html和js写到一块去了，
而tailwind css 让css也跟html，js糅合到一块去。这种把html，css写在JavaScript的方式可能有人不喜欢，没关系，
反正前端框架到处都是，分开写的也有很多。

## 优化字体

`/app/ui/`目录里创建`fonts.ts`文件,写入：
```javascript
import { Inter, Lusitana } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });

export const lusitana = Lusitana({
    weight: ['400', '700'],
    subsets: ['latin'],
});
```
导入google字体。

`layout.ts`文件里写入：
```javascript
import '@/app/ui/global.css';
import {inter} from '@/app/ui/fonts';

export default function RootLayout({
                                       children,
                                   }: {
    children: React.ReactNode;
}) {
    return (
        <html lang="en">
        <body className={`${inter.className} antialiased`}>{children}</body>
        </html>
    );
}
```
使字体对整个网页生效，antialiased是一个tailwind类名，作用是使字体平滑。
到`page.tsx`文件里取消`<AcmeLogo/>`组件的注释。

## 优化图片
nextjs的图片的根目录为`/public`，在这里放图片。
用nextjs的`<Image>`组件替代html的`<img>`标签，可以响应不同屏幕图像的尺寸。

在page.tsx里写入
```javascript
import Image from "next/image";
```
导入组件。
然后在return里头写入：
```javascript
<Image
    src="/hero-desktop.png"
    width={1000}
    height={760}
    className="hidden md:block"
    alt="Screenshots of the dashboard project showing desktop version"
/>
<Image
    src="/hero-mobile.png"
    width={560}
    height={620}
    className="block md:hidden"
    alt="Screenshot of the dashboard project showing mobile version"
/>
```
做好后效果图如下：
<img src="https://images2.imgbox.com/cb/8c/N563eHYh_o.png" alt="image host"/>
