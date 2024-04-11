---
title: next教程（四）
date: 2024-04-06 18:41:17
tags:
---

## 动态渲染
对那些需要实时更新的页面采用动态渲染，在`data.tsx`里写入：
```javascript
import { unstable_noStore as noStore } from 'next/cache';
```
导入unstable_noStore这个函数，然后在fetchRevenue ，fetchLatestInvoices，fetchCardData，fetchFilteredInvoices，fetchInvoiceById，fetchFilteredCustomers这几个函数里调用。

## 流传输
动态渲染带来的问题就是，应用会等待最慢的那个函数获取数据。然后**一次性**显示。流传输就是让它分解为块，一块一块的传输。
整个页面流传输，需要创建一个`loading.tsx`文件，比如创建一个`app/dashboard/loading.tsx`,写入：
```javascript
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
    return <DashboardSkeleton />;
}
```
在dashboard目录内创建这个文件将会影响所有页面，这时候可以使用路由分组，就是新建一个名字带括号的目录。比如新建
`app\dashboard\(overview)\`目录,把`loading.tsx`,`page.tsx`放到这个目录里。

组件流传输：参考react的suspend组件用法。

## 发票页面添加搜索
在`app/dashboard/invoices/page.tsx`文件中写入：
```javascript
import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import {CreateInvoice} from '@/app/ui/invoices/buttons';
import {lusitana} from '@/app/ui/fonts';
import {InvoicesTableSkeleton} from '@/app/ui/skeletons';
import {Suspense} from 'react';

export default async function Page({
                                       searchParams,
                                   }: {
    searchParams?: {
        query?: string;
        page?: string;
    };
}) {

    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

    return (
        <div className="w-full">
            <div className="flex w-full items-center justify-between">
                <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
            </div>
            <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
                <Search placeholder="Search invoices..."/>
                <CreateInvoice/>
            </div>
            <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton/>}>
                <Table query={query} currentPage={currentPage}/>
            </Suspense>
            <div className="mt-5 flex w-full justify-center">
                {/* <Pagination totalPages={totalPages} /> */}
            </div>
        </div>
    );
}
```
在`app/ui/search.tsx`文件中写入：
```javascript
'use client';

import {MagnifyingGlassIcon} from '@heroicons/react/24/outline';
import {useSearchParams,usePathname, useRouter} from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';


export default function Search({placeholder}: { placeholder: string }) {
    const searchParams = useSearchParams();
    const pathname = usePathname();
    const { replace } = useRouter();

    const handleSearch = useDebouncedCallback((term) => {
        console.log(`Searching... ${term}`);

        const params = new URLSearchParams(searchParams);
        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }
        replace(`${pathname}?${params.toString()}`);
    }, 300);

    return (
        <div className="relative flex flex-1 flex-shrink-0">
            <label htmlFor="search" className="sr-only">
                搜索
            </label>
            <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                placeholder={placeholder}
                onChange={(e) => {
                    handleSearch(e.target.value);
                }}
                defaultValue={searchParams.get('query')?.toString()}
            />
            <MagnifyingGlassIcon
                className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900"/>
        </div>
    );
}

```

## 添加分页
`app/ui/invoices/pagination.tsx`文件中写入：
```javascript
'use client';

import {ArrowLeftIcon, ArrowRightIcon} from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import {generatePagination} from '@/app/lib/utils';
import {usePathname, useSearchParams} from 'next/navigation';


export default function Pagination({totalPages}: { totalPages: number }) {

    const pathname = usePathname();
    const searchParams = useSearchParams();
    const currentPage = Number(searchParams.get('page')) || 1;

    const allPages = generatePagination(currentPage, totalPages);

    const createPageURL = (pageNumber: number | string) => {
        const params = new URLSearchParams(searchParams);
        params.set('page', pageNumber.toString());
        return `${pathname}?${params.toString()}`;
    };


    return (
        <>
            <div className="inline-flex">
                <PaginationArrow
                    direction="left"
                    href={createPageURL(currentPage - 1)}
                    isDisabled={currentPage <= 1}
                />

                <div className="flex -space-x-px">
                    {allPages.map((page, index) => {
                        let position: 'first' | 'last' | 'single' | 'middle' | undefined;

                        if (index === 0) position = 'first';
                        if (index === allPages.length - 1) position = 'last';
                        if (allPages.length === 1) position = 'single';
                        if (page === '...') position = 'middle';

                        return (
                            <PaginationNumber
                                key={page}
                                href={createPageURL(page)}
                                page={page}
                                position={position}
                                isActive={currentPage === page}
                            />
                        );
                    })}
                </div>

                <PaginationArrow
                    direction="right"
                    href={createPageURL(currentPage + 1)}
                    isDisabled={currentPage >= totalPages}
                />
            </div>
        </>
    );
}

function PaginationNumber({
                              page,
                              href,
                              isActive,
                              position,
                          }: {
    page: number | string;
    href: string;
    position?: 'first' | 'last' | 'middle' | 'single';
    isActive: boolean;
}) {
    const className = clsx(
        'flex h-10 w-10 items-center justify-center text-sm border',
        {
            'rounded-l-md': position === 'first' || position === 'single',
            'rounded-r-md': position === 'last' || position === 'single',
            'z-10 bg-blue-600 border-blue-600 text-white': isActive,
            'hover:bg-gray-100': !isActive && position !== 'middle',
            'text-gray-300': position === 'middle',
        },
    );

    return isActive || position === 'middle' ? (
        <div className={className}>{page}</div>
    ) : (
        <Link href={href} className={className}>
            {page}
        </Link>
    );
}

function PaginationArrow({
                             href,
                             direction,
                             isDisabled,
                         }: {
    href: string;
    direction: 'left' | 'right';
    isDisabled?: boolean;
}) {
    const className = clsx(
        'flex h-10 w-10 items-center justify-center rounded-md border',
        {
            'pointer-events-none text-gray-300': isDisabled,
            'hover:bg-gray-100': !isDisabled,
            'mr-2 md:mr-4': direction === 'left',
            'ml-2 md:ml-4': direction === 'right',
        },
    );

    const icon =
        direction === 'left' ? (
            <ArrowLeftIcon className="w-4"/>
        ) : (
            <ArrowRightIcon className="w-4"/>
        );

    return isDisabled ? (
        <div className={className}>{icon}</div>
    ) : (
        <Link className={className} href={href}>
            {icon}
        </Link>
    );
}

```
然后取消page.tsx文件中的pagination组件注释。在`app/ui/search.tsx`文件中，handleSearch函数里写入：
```javascript
params.set('page', '1');
```

最后效果：
<img src="https://images2.imgbox.com/21/b1/A42Su9rE_o.png" alt="image host"/>