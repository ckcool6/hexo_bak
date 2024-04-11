---
title: next教程（三）
date: 2024-04-05 14:42:09
tags:
---
## 创建数据库
web程序大部分都是从数据库里获取一些数据，然后显示出来。有了页面，但是页面上还没有数据。首先，需要先要有个数据库，数据库可以是本地的可以是远程的，这里采用远程数据库。[点击此链接](https://nextjs.org/learn/dashboard-app/setting-up-your-database) 创建一个远程数据库，前提是得有GitHub账号和Vercel 账号。

创建完之后把远程数据库的`.env.local`文件中的内容复制到项目根目录，重命名为`.env`。
然后运行 `npm install @vercel/postgres` 安装Vercel Postgres SDK。

## 添加一些数据
在package.json文件里写入：
```json
"scripts": {
  "build": "next build",
  "dev": "next dev",
  "start": "next start",
  "seed": "node -r dotenv/config ./scripts/seed.js"
},
```
然后运行`npm run seed`,即可把预先准备的数据添加进去。

## 获取数据
打开`/app/lib/data.ts`文件，在这个文件里查询数据，先写入：
```javascript
import { sql } from '@vercel/postgres';
```
导入sql函数。然后打开`/app/dashboard/page.tsx`文件，写入：
```javascript
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
 
export default async function Page() {
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        {/* <Card title="Collected" value={totalPaidInvoices} type="collected" /> */}
        {/* <Card title="Pending" value={totalPendingInvoices} type="pending" /> */}
        {/* <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> */}
        {/* <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> */}
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        {/* <RevenueChart revenue={revenue}  /> */}
        {/* <LatestInvoices latestInvoices={latestInvoices} /> */}
      </div>
    </main>
  );
}
```
在这个文件里，page组件前面有个async，是异步组件。导入了三个组件来接受数据： `<Card>`, `<RevenueChart>`,  `<LatestInvoices>`。
接下来在此文件里写入：
```javascript
import { fetchRevenue } from '@/app/lib/data';
```
导入fetchRevenue函数，在page函数里写入：
```javascript
const revenue = await fetchRevenue();
```
调用此函数。然后取消`<RevenueChart>`组件的注释。打开浏览器`127.0.0.1:3000/dashboard`查看效果。

## 更多数据
打开`app/lib/data.ts`,写入:
```javascript
// Fetch the last 5 invoices, sorted by date
export async function fetchLatestInvoices() {
    try {
        const data = await sql<LatestInvoiceRaw>`
      SELECT invoices.amount, customers.name, customers.image_url, customers.email, invoices.id
      FROM invoices
      JOIN customers ON invoices.customer_id = customers.id
      ORDER BY invoices.date DESC
      LIMIT 5`;

        const latestInvoices = data.rows.map((invoice) => ({
            ...invoice,
            amount: formatCurrency(invoice.amount),
        }));
        return latestInvoices;
    } catch (error) {
        console.error('Database Error:', error);
        throw new Error('Failed to fetch the latest invoices.');
    }
}
```

打开 `page.tsx` 文件 ，然后写入：
```javascript
import {fetchRevenue, fetchLatestInvoices} from '@/app/lib/data';
```

page函数内写入：
```javascript
const latestInvoices = await fetchLatestInvoices();
```
然后取消组件的注释，查看效果。

最后获取一个还剩一个card组件，修改`page.tsx`:
```javascript
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import {
  fetchRevenue,
  fetchLatestInvoices,
  fetchCardData,
} from '@/app/lib/data';
 
export default async function Page() {
  const revenue = await fetchRevenue();
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();
 
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Card title="Collected" value={totalPaidInvoices} type="collected" />
        <Card title="Pending" value={totalPendingInvoices} type="pending" />
        <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
        <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        />
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <RevenueChart revenue={revenue} />
        <LatestInvoices latestInvoices={latestInvoices} />
      </div>
    </main>
  );
}
```

最终效果：
<img src="https://images2.imgbox.com/5e/2a/BxQ7rWhj_o.png" alt="image host"/>