---
title: 译文：每个开发者都应该了解的基本 React Query模式
author: 渡一前端每日精选
date: 
url: https://mp.weixin.qq.com/s?__biz=MzI2NTQ5NTE4OA==&mid=2247565438&idx=1&sn=c9802493a9dd72072df6f7e151d3c251&chksm=eb7960852c5e6e26aade416f55c655b14008f249b2ae67c29c3489cec8a7ce1a1f3b3dad84ef&mpshare=1&scene=24&srcid=0604eBLKlJVVyxvtMY5L4sTO&sharer_shareinfo=40775f9322c35dec83b81ce8751ddce5&sharer_shareinfo_first=40775f9322c35dec83b81ce8751ddce5#rd
---

> 原文地址：https://chamith.medium.com/essential-react-query-patterns-every-developer-should-know-8870e9ad391c
>
> 原文作者： Chamith Madusanka

使用 `TanStack Query` 构建可扩展、可维护 React 应用的完整指南。

`React Query`（现更名为 `TanStack Query`）彻底改变了我们在 React 应用中处理 server state（服务端状态）的方式。但正如任何强大的工具一样，掌握正确的模式，才是在与库较劲和充分发挥其潜力之间的关键区别。

在多个生产应用中使用 `React Query` 的过程中，我总结出了一套在架构良好的代码库中反复出现的模式。这些并非只是理论上的最佳实践——它们是经过实战验证的解决方案，能消除 bug、提升性能、加速开发。

让我们深入这些必知模式。

## 从基础开始：Query 的解剖

每个 `React Query` 实现都从这里出发。在进一步构建之前，理解这个基础至关重要。

```
import { useQuery } from '@tanstack/react-query';import { getContacts } from './api/client';  
function ContactsTable() {  const { data, isPending, isError, refetch } = useQuery({    queryKey: ['contacts'],    queryFn: getContacts,  });  if (isPending) return <LoadingSpinner />;  if (isError) return <ErrorAlert onRetry={refetch} />;  
  return <Table data={data} />;}
```

queryKey 不只是一个标签——它是 `React Query` 内部的缓存标识符。这个简单的数组解锁了该库最强大的特性之一：智能缓存与自动后台重新获取（background refetching）。

但这只是开始。

## Pattern 1：Custom Query Hooks

随着代码库的增长，你会发现自己在重复相同的 query 配置。解决方案？将它们提取为 custom hooks。

```
// queries/contacts.tsexport function useContacts() {  return useQuery({    queryKey: ['contacts'],    queryFn: getContacts,  });}  
// Componentfunction ContactsTable() {  const { data, isPending, isError } = useContacts();  // 简洁、聚焦的组件逻辑}
```

这个模式让组件专注于呈现，同时将数据获取的关注点集中管理。代码因此变得更易读、更易维护。

但我们还可以做得更好。

## Pattern 2：Query Options 实现最大灵活性

有趣的地方来了。与其创建直接调用 `useQuery` 的 custom hooks，不如创建 query option 对象：

```
import { queryOptions } from '@tanstack/react-query';  
export const contactsQueryOptions = queryOptions({  queryKey: ['contacts'],  queryFn: getContacts,});
```

为什么这样更好？有两个原因：

第一，完美的 TypeScript 类型推断。你不需要手动定义类型——它们会自动流入。

第二，极强的灵活性。你可以直接使用这些 options，也可以对它们进行扩展：

```
// 直接使用function ContactsList() {  const { data } = useQuery(contactsQueryOptions);  return <List items={data} />;}  
// 配合 custom selectors 扩展使用function ContactsCount() {  const { data } = useQuery({    ...contactsQueryOptions,    select: (contacts) => contacts.length,  });  return <Badge count={data} />;}
```

随着应用规模扩大，这一点将变得无比宝贵。

## Pattern 3：Selectors 实现性能优化

`selectors` 的作用不止于转换数据——它还是一个许多开发者忽视的性能优化工具。

```
const contactsQueryOptions = queryOptions({  queryKey: ['contacts'],  queryFn: getContacts,  select: (data) => data.length,});
```

关键洞察：如果你只关心联系人数量，而后端某个联系人的名称发生了变化，你的组件不会重新渲染。`selector` 比较的是转换后的结果，而非原始数据。

在企业级规模下，这种差异巨大。`React Query` 会智能地阻止不必要的重新渲染，而不是让数十次无意义的 re-render 在组件树中级联传播。

## Pattern 4：参数化 Queries

真实应用需要动态 query。以下是正确的处理方式：

```
export const contactQueryOptions = (contactId: string) =>   queryOptions({    queryKey: ['contacts', contactId],    queryFn: () => getContact(contactId),  });  
// 使用function ContactPage() {  const { id } = useParams();  const { data } = useQuery(contactQueryOptions(id));  return <ContactDetails contact={data} />;}
```

**关键规则：始终将所有参数包含在 `queryKey` 中。** 如果你在 key 中遗漏了 `contactId`，`React Query` 可能会返回另一个联系人的缓存数据。这是我在生产应用中最常见的 bug 之一。

## Pattern 5：轻松实现分页

使用 `React Query`，分页出奇地简单——它不过是一个带 state 的参数化 query：

```
export const paginatedContactsOptions = (page: number, pageSize: number) =>  queryOptions({    queryKey: ['contacts', 'paginated', page, pageSize],    queryFn: () => getContacts({ page, pageSize }),  });  
function ContactsTable() {  const [page, setPage] = useState(1);  const { data } = useQuery(paginatedContactsOptions(page, 10));  return (    <>      <Table data={data.items} />      <Pagination         currentPage={page}         onNext={() => setPage(p => p + 1)}       />    </>  );}
```

`React Query` 会在 `page` 变化时自动重新获取数据。无需手动干预，无需 `useEffect` 体操——它就是能用。

## Pattern 6：Prefetching 实现零 Loading 状态

想彻底消除 loading spinner？预获取用户很可能需要的数据：

```
function ContactsTable() {  const [page, setPage] = useState(1);  const queryClient = useQueryClient();  const { data } = useQuery(paginatedContactsOptions(page, 10));  
  useEffect(() => {    // 在后台静默加载下一页    queryClient.prefetchQuery(      paginatedContactsOptions(page + 1, 10)    );  }, [page, queryClient]);  return <Table data={data} />;}
```

当用户点击"下一页"时，数据已经就绪。体验感觉是即时的，因为它确实是即时的。

你可以预获取多页、在 hover 时预获取，或根据用户行为模式预获取。可能性无穷无尽。

## Pattern 7：Infinite Queries 实现无限滚动

社交媒体风格的无限滚动已内置于 `React Query`：

```
export const infiniteContactsOptions = queryOptions({  queryKey: ['contacts', 'infinite'],  queryFn: ({ pageParam }) => getContacts({ cursor: pageParam }),  initialPageParam: undefined,  getNextPageParam: (lastPage) => lastPage.nextCursor,});  
function InfiniteContactsList() {  const {     data,     fetchNextPage,     isFetchingNextPage   } = useInfiniteQuery(infiniteContactsOptions);  return (    <>      {data.pages.map(page =>         page.items.map(contact => (          <ContactCard key={contact.id} {...contact} />        ))      )}      <button onClick={() => fetchNextPage()}>        {isFetchingNextPage ? 'Loading...' : 'Load More'}      </button>    </>  );}
```

`React Query` 在内部管理所有 cursor state。你只需定义如何从每次响应中提取下一个 cursor，其余的它来处理。

## Pattern 8：Query Key Factories

随着应用的增长，手动重建 query key 会变得容易出错且繁琐。`Query key factories` 优雅地解决了这个问题：

```
export const contactKeys = {  all: ['contacts'] as const,  lists: () => [...contactKeys.all, 'list'] as const,  list: (filters: ContactFilters) =>     [...contactKeys.lists(), filters] as const,  details: () => [...contactKeys.all, 'detail'] as const,  detail: (id: string) =>     [...contactKeys.details(), id] as const,};  
// 在 queries 中使用export const contactQueryOptions = (id: string) =>   queryOptions({    queryKey: contactKeys.detail(id),    queryFn: () => getContact(id),  });  
// 精准的缓存失效queryClient.invalidateQueries({   queryKey: contactKeys.all }); // 使所有缓存失效queryClient.invalidateQueries({   queryKey: contactKeys.lists() }); // 仅使 list queries 失效
```

这种层级结构让缓存管理变得可预测且易于维护。再也不用担心 query key 是否写对了。

## Pattern 9：Mutations 用于修改数据

读取数据只是等式的一半，修改数据是另一半：

```
export function useDeleteContact() {  return useMutation({    mutationFn: (contactId: string) => deleteContact(contactId),    onSuccess: () => {      toast.success('Contact deleted successfully');    },    onError: () => {      toast.error('Failed to delete contact');    },  });}  
// 在组件中使用function ContactCard({ contact }) {  const { mutate, isPending } = useDeleteContact();  return (    <Card>      <h3>{contact.name}</h3>      <button         onClick={() => mutate(contact.id)}        disabled={isPending}      >        {isPending ? 'Deleting...' : 'Delete'}      </button>    </Card>  );}
```

生命周期回调（`onSuccess`、`onError`、`onSettled`）让你精确控制每个阶段的行为。

## Pattern 10：自动 Query 失效

Mutations 执行后，相关 queries 需要刷新。不要在各处重复这段逻辑——将其自动化：

```
// 在你的 mutation 中export function useDeleteContact() {  return useMutation({    mutationFn: (contactId: string) => deleteContact(contactId),    meta: {      invalidates: [contactKeys.all],    },  });}  
// 全局配置（一次性，在 main.tsx 中）const queryClient = new QueryClient({  defaultOptions: {    mutations: {      onSettled: async (data, error, variables, context) => {        const meta = context?.meta;        if (meta?.invalidates) {          await Promise.all(            meta.invalidates.map((queryKey) =>              queryClient.invalidateQueries({ queryKey })            )          );        }      },    },  },});
```

现在每个 mutation 都会自动使相应的 queries 失效。无需重复代码，无遗漏失效，无过期数据。

## Pattern 11：全局错误处理

一次性全局处理常见错误：

```
const queryClient = new QueryClient({  defaultOptions: {    mutations: {      onError: (error) => {        // 全局处理身份验证错误        if (error.status === 401) {          logout();          navigate('/login');        }  
        // 处理网络错误        if (error.message === 'Network Error') {          toast.error('Check your connection');        }      },    },  },});
```

每个 mutation 都自动受益于此错误处理。无需在数十个 mutations 中重复身份验证逻辑。

## Pattern 12：Optimistic Updates

在服务器响应之前就更新 UI，让界面感觉瞬间响应。有两种方式：

### UI 级别（更简单）

```
function useContactsBeingDeleted() {  return useMutationState({    filters: {       mutationKey: ['deleteContact'],       status: 'pending'     },    select: (mutation) => mutation.state.variables,  });}  
function ContactsList() {  const { data: contacts } = useContacts();  const deletingIds = useContactsBeingDeleted();  
  // 过滤掉正在被删除的联系人  const visibleContacts = contacts.filter(    c => !deletingIds.includes(c.id)  );  
  return <List items={visibleContacts} />;}
```

### 缓存级别（更健壮）

```
export function useDeleteContact() {  const queryClient = useQueryClient();  
  return useMutation({    mutationFn: deleteContact,    onMutate: async (contactId) => {      // 取消正在进行的重新获取      await queryClient.cancelQueries({         queryKey: contactKeys.lists()       });  
      // 快照当前值      const previousContacts = queryClient.getQueryData(        contactKeys.lists()      );      // 乐观更新      queryClient.setQueryData(        contactKeys.lists(),        (old) => old.filter(c => c.id !== contactId)      );      // 返回回滚数据      return { previousContacts };    },    onError: (err, variables, context) => {      // 出错时回滚      if (context?.previousContacts) {        queryClient.setQueryData(          contactKeys.lists(),          context.previousContacts        );      }    },    onSettled: () => {      // 始终重新获取以保证一致性      queryClient.invalidateQueries({         queryKey: contactKeys.lists()       });    },  });}
```

缓存级别的方式会自动在应用的所有地方更新数据。任何使用联系人数据的组件都能立即看到乐观更新的结果。

## Pattern 13：Suspense Queries

通过 React `Suspense` 消除分散的 loading 状态：

```
// 将 useQuery 替换为 useSuspenseQueryfunction ContactsList() {  const { data } = useSuspenseQuery(contactsQueryOptions);  // 不再需要 isPending 检查！  return <Table data={data} />;}  
function ContactDetails({ id }) {  const { data } = useSuspenseQuery(contactQueryOptions(id));  return <Details contact={data} />;}  
// 集中式 loading UIfunction App() {  return (    <Suspense fallback={<AppSkeleton />}>      <ContactsList />      <ContactDetails id="123" />    </Suspense>  );}
```

不再是 loading spinner 在应用中此起彼伏地闪烁，而是获得一个统一的 loading 体验。UI 感觉更精致、更专业。

## 融会贯通

以下是将这些模式组合后，一个生产就绪的 `React Query` 配置：

```
// queries/contacts.tsexport const contactKeys = {  all: ['contacts'] as const,  lists: () => [...contactKeys.all, 'list'] as const,  list: (filters: Filters) => [...contactKeys.lists(), filters] as const,  detail: (id: string) => [...contactKeys.all, id] as const,};  
export const contactsQueryOptions = (filters: Filters) =>  queryOptions({    queryKey: contactKeys.list(filters),    queryFn: () => getContacts(filters),  });  
export function useDeleteContact() {  const queryClient = useQueryClient();  
  return useMutation({    mutationFn: deleteContact,    mutationKey: ['deleteContact'],    meta: { invalidates: [contactKeys.all] },    onMutate: async (contactId) => {      await queryClient.cancelQueries({         queryKey: contactKeys.lists()       });  
      const previous = queryClient.getQueryData(        contactKeys.lists()      );  
      queryClient.setQueryData(        contactKeys.lists(),        (old) => old?.filter(c => c.id !== contactId)      );  
      return { previous };    },    onError: (err, variables, context) => {      if (context?.previous) {        queryClient.setQueryData(          contactKeys.lists(),           context.previous        );      }    },  });}  
// components/ContactsList.tsxfunction ContactsList() {  const [page, setPage] = useState(1);  const queryClient = useQueryClient();  
  const { data } = useSuspenseQuery(    contactsQueryOptions({ page, pageSize: 20 })  );  
  const { mutate: deleteContact } = useDeleteContact();  
  // 预获取下一页  useEffect(() => {    queryClient.prefetchQuery(      contactsQueryOptions({ page: page + 1, pageSize: 20 })    );  }, [page, queryClient]);  
  return (    <Table      data={data.items}      onDelete={deleteContact}      pagination={{ page, onChange: setPage }}    />  );}
```

这段代码具备：

* 全程类型安全
* 通过优化 re-render 实现高性能
* 借助 prefetching 和 optimistic updates 提供良好的用户体验
* 通过集中式逻辑和一致的模式保持可维护性
* 具备自动错误处理和缓存管理的健壮性

## 核心原则总结

* **始终将所有参数包含在 query key 中**，防止缓存冲突
* **优先使用 query options 而非 custom hooks**，获得更好的 TypeScript 支持和灵活性
* **善用 selectors** 在规模化时阻止不必要的 re-render
* **尽早引入 query key factories**，保持一致性
* **通过 metadata 自动化 query 失效**，减少样板代码
* **考虑 optimistic updates**，提升用户感知性能
* **使用 Suspense queries** 简化 loading state 管理

## 写在最后

`React Query` 不只是一个数据获取库——它是 server state 管理的完整解决方案。这些模式凝聚了无数生产应用的集体智慧，以及数千小时真实世界经验的结晶。

你不必一次性实现所有这些模式。从基础开始，随着应用的成长逐步添加模式，然后看着你的代码库变得更简洁、更快速、更易维护。

`React Query` 的美妙之处在于它与你共同成长。从最初简单的 `useQuery` 调用，演进为一个成熟的、高性能的数据层，处理缓存、prefetching、optimistic updates 等诸多功能——而这一切只需极少的样板代码。

掌握这些模式，你将永远以不同的眼光看待数据获取。

RECOMMEND

推荐阅读