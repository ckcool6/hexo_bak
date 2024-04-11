---
title: next教程（二）
date: 2024-04-04 19:10:38
tags:
---

## 路由

nextjs路由采用文件路径映射，比如新建一个`app/dashboard/page.tsx`,url就是`127.0.0.1:3000/dashboard`。
新建`app/dashboard/page.tsx`文件，写入：

```javascript
export default function Page() {
    return <p>仪表盘</p>;
}
```

在浏览器地址栏输入`127.0.0.1:3000/dashboard`查看，显示刚创建好的页面。

新建`app/dashboard/customers/page.tsx`和`app/dashboard/invoices/page.tsx`文件，分别写入：

```javascript
export default function Page() {
    return <p>客户页面</p>;
}
```

```javascript
export default function Page() {
    return <p>发票页面</p>;
}
```

在地址栏输入`127.0.0.1:3000/dashboard/customers`, `127.0.0.1:3000/dashboard/invoices`查看。

## 修改布局

建好的页面没有布局，只显示几个字。创建布局的方法是在文件夹里新建`layout.tsx`文件，比如在dashboard文件夹里创建
`layout.tsx`，就是这个文件夹里所有的页面都采用这个布局。

创建`app/dashboard/layout.tsx`文件，写入：

```javascript
import SideNav from '@/app/ui/dashboard/sidenav';

export default function Layout({children}: { children: React.ReactNode }) {
    return (
        <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
            <div className="w-full flex-none md:w-64">
                <SideNav/>
            </div>
            <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
        </div>
    );
}
```
来应用布局。

## 页面跳转

用next的`<Link>`组件来替代`<a>`标签,这样就可以不用刷新整个页面，找到`app/ui/dashboard/nav-link.tsx`文件，写入

```javascript
'use client'; //客户端组件，用到hook的时候加上这句

import {
    UserGroupIcon,
    HomeIcon,
    DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link'; //导入link组件
import {usePathname} from 'next/navigation'; //导入hook
import clsx from 'clsx';

// Map of links to display in the side navigation.
// Depending on the size of the application, this would be stored in a database.
const links = [
    {name: '主页', href: '/dashboard', icon: HomeIcon},
    {
        name: '发票',
        href: '/dashboard/invoices',
        icon: DocumentDuplicateIcon,
    },
    {name: '客户', href: '/dashboard/customers', icon: UserGroupIcon},
];

export default function NavLinks() {
    const pathname = usePathname();
    return (
        <>
            {links.map((link) => {
                const LinkIcon = link.icon;
                return (
                    <Link
                        key={link.name}
                        href={link.href}
                        className={clsx(
                            'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
                            {
                                'bg-sky-100 text-blue-600': pathname === link.href,
                            },
                        )}
                    >
                        <LinkIcon className="w-6"/>
                        <p className="hidden md:block">{link.name}</p>
                    </Link>
                );
            })}
        </>
    );
}
```

效果图：
<img src="https://images2.imgbox.com/b5/e7/W4P3wSDw_o.png" alt="image host"/>

