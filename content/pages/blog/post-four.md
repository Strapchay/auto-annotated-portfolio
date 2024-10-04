---
type: PostLayout
title: Manually Updating Pages in React-query’s useInfiniteQuery
colors: colors-a
date: '2024-05-27'
author: content/data/team/doris-soto.json
excerpt: More context that may or may not be helpful
featuredImage:
  type: ImageBlock
  url: /images/featured-Image4.jpg
  altText: Post thumbnail image
bottomSections:
  - elementId: ''
    type: RecentPostsSection
    colors: colors-f
    variant: variant-d
    subtitle: Recent posts
    showDate: true
    showAuthor: false
    showExcerpt: true
    recentCount: 2
    styles:
      self:
        height: auto
        width: wide
        margin:
          - mt-0
          - mb-0
          - ml-0
          - mr-0
        padding:
          - pt-12
          - pb-56
          - pr-4
          - pl-4
        justifyContent: center
      title:
        textAlign: left
      subtitle:
        textAlign: left
      actions:
        justifyContent: center
    showFeaturedImage: true
    showReadMoreLink: true
  - type: ContactSection
    backgroundSize: full
    colors: colors-f
    styles:
      self:
        height: auto
        width: narrow
        margin:
          - mt-0
          - mb-0
          - ml-4
          - mr-4
        padding:
          - pt-24
          - pb-24
          - pr-4
          - pl-4
        alignItems: center
        justifyContent: center
        flexDirection: row
      title:
        textAlign: left
      text:
        textAlign: left
---
When working with remote state management in a react project, react-query is one of the available options to go for and my technology of choice for this use-case.

Working with react-query’s `useInfinteQuery` hook comes into use when enhancing the user’s experience in terms of relation to paginated data. The classic approach is to render a pagination navigator to move through each and specific resources. However, based on specific use-cases which might include loading a chat list or serving contents in the case of social networks like Instagram and the likes, useInfiniteQuery is a great hook.

In my case, i had to work on a chat application which required listing both the previous conversations of the user and also fetch newly updated data to keep the UI state in sync with the remote state(DB). Fair enough, react-query provides a plug-and-play interface to perform most use-cases in relation to this functionality, automatically fetching both previous and next page depending on the method called and based on your outlined logic within the `getNextPageParams` or the `getPrevPageParams` options. It also provides a `refetch`option to re-fetch all pages contained within the infiniteQueries result which it performs by fetching sequentially as according to their docs [here](https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries#what-happens-when-an-infinite-query-needs-to-be-refetched.). However, when it comes to having to re-fetch a specific page and keeping the rest of the data stale, you would have to drop down to manually mutating and implementing your desired logic which is what this write-up is meant to address.

In my case, using a websocket as the base logic for the chat interactions, means i can always get updated data as responses, this gives an ability to access the updated data in real-time but it doesn’t bridge the gap of making the api data held within the inifiniteQueries pages up-to-date except those data are immediately invalidated on each new data pushed in or gotten from the websocket connection. Depending on philosophies, you might prefer re-fetching all the pages, but i would differ on that. Keeping state of the data sent and gotten from the websocket would be a more resource focused approach however, it does introduce the complexity of syncing the data with the inifiniteQueries result and with this, manually fetching the latest page and adding to the infiniteQueries results would be a better approach, as the already fetched data in the infiniteQueries needs to be Stale for pages the user already fetched.

Following from my preferred approach, when the websocket state data is the same as the amount of data returned per-page, the last page cached within the hook is then fetched and pushed into the inifiniteQueries pages, which would mean, replacing the page data at that index with the updated one and then refetching the latest page available to account for any difference in state. To implement this, the following hook was used:

```
export function useInvalidateResourceLastPage(dataState) {
 const queryClient = useQueryClient();
 const { resourceId } = useParams();
 const lastCachedPageNum = useRef(null);
 const [invalidatePage, setInvalidatePage] = useState(false); //state to trigger refetching
 const [dataRefreshed, setDataRefreshed] = useState(false); //external state to notify the component using the hook fresh data is available

 const {
   isPending: isLoading,
   data: updatedData,
   status,
   refetch,
 } = useQuery({
   queryKey: [“resourceName”, { page: lastCachedPageNum.current }],
   queryFn: () =>
   getResource({ pageParam: lastCachedPageNum.current, resourceId }),
   enabled: !!lastCachedPageNum.current,
 });

 useEffect(() => {
   if (invalidatePage) {
     const lastCachedPage = dataState?.pages[dataState?.pages.length — 1];
     const newCachedLastPage = getLastCachedPageNum(lastCachedPage);
     setDataRefreshed((_) => false);
      if (newCachedLastPage === lastCachedPageNum.current) refetch(); 
//if the cached page to fetch is same as the last cached page, then refetch the useQuery data else set the last cached page as that value
    else lastCachedPageNum.current = newCachedLastPage;
 }
 }, [invalidatePage, dataState, queryClient, refetch]);

useEffect(() => {
   if (!isLoading && status === “success” && updatedData) {
   queryClient.setQueryData([“resourceName”, resourceId], (data) =>
     updateQueryData(data, “add”, lastCachedPageNum.current, updatedData),
   );
   setDataRefreshed((_) => true);
   setInvalidatePage((_) => false);
   }
 }, [
 resourceId,
 isLoading,
 status,
 updatedData,
 queryClient,
 lastCachedPageNum,
 ]);

function invalidateLastQuery() {
 setInvalidatePage((_) => true);
 }
} 
return { invalidateLastQuery, dataRefreshed };
}
```

From the above code, the resourceId being the id of the current chat in question. The `invalidateLastQuery` is used and exposed to the component using this hook to invalidate the last cached page which is the last page contained in the infiniteQueries page, after which the first use effect is executed and based on that value gotten and then it gets the value of the page to invalidate using the `getLastCachedPageNum` function which would be based on the logic you are using to get the page to be invalidated, the function checks the page’s url to determine what the current page is since the api only returns the next and previous pages value in its url. The function looks as follows:

```
export function getLastCachedPageNum(cachedPage) {
 let pageNum = 0;
 let nextNum = Number(
   cachedPage?.next ? getUrlPageQuery(cachedPage?.next) : 0,
 );
 let prevNum = Number(
   cachedPage?.previous ? getUrlPageQuery(cachedPage.previous) : 0,
 );
 if (nextNum === 0 && prevNum !== 0) pageNum = prevNum += 1;
 if (prevNum === 0 && nextNum !== 0) pageNum = nextNum -= 1;
 if (cachedPage?.previous && prevNum === 0 && nextNum === 0) pageNum = 2;
 if (!cachedPage?.previous && prevNum === 0 && nextNum === 0) pageNum = 1;
 return pageNum;
}
```

The second use effect is triggered, which then mutates the infiniteQueries data with the newly fetched one, using the `updateQueryData` function which is where the mutation logic resides. That function looks as follows:

```
function updateQueryData(dataState, updateType, pageNum, updateData = null) {
   if (updateType === “add” && pageNum && updateData) {
     const queryWithMatchingIndex = dataState?.pages.findIndex((page) =>
       page.previous === updateData.previous && page.next === updateData.next,
     );
     let updateState = {};
    if (queryWithMatchingIndex > -1) {
     updateState = {
       pages: dataState.pages.toSpliced(
         queryWithMatchingIndex,
         queryWithMatchingIndex + 1,
         updateData,
       ),
       pageParams:
         dataState?.pages.length === 1
         ? [pageNum]
         : […dataState.pageParams, pageNum],
     };
   } else if (queryWithMatchingIndex === -1 && dataState?.pages.length > 1) {
     updateState = {
       pages: [
         …dataState.pages.slice(0, dataState?.pages.length — 1),
         updateData,
       ],
       pageParams: [
         …dataState.pageParams.slice(0, dataState?.pageParams.length — 1),
         pageNum,
       ],
     };
   } else if (queryWithMatchingIndex === -1 && dataState?.pages.length === 1) {
     updateState = {
       pages: [updateData],
       pageParams: [pageNum],
     };
   }
 return updateState;
 }
}
```

The above function basically, checks the index passed in and based on that, it mutates the pages to replace it with the updated page as needed. The doc example can be found [here](https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries#manually-removing-first-page). This should be able to address your needs relating to manual mutation if you are ever faced with an edge-case or implementation requirement as in my case or that fits implementing such logic.

Here is a link to my [github](https://github.com/strapchay), if you would like to follow.
