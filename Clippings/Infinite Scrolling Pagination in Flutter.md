---
title: "Infinite Scrolling Pagination in Flutter"
source: "https://www.kodeco.com/14214369-infinite-scrolling-pagination-in-flutter/page/2"
author:
  - "[[ebueno]]"
published:
created: 2026-03-17
description: "Learn how to implement infinite scrolling pagination (also known as lazy loading) in Flutter using the Infinite Scroll Pagination package."
tags:
  - "clippings"
---
This article has been archived and is no longer being updated. It may not work with the most recent OS versions.

You are currently viewing page 2 of 3 of this article. [Click here to view the first page](https://www.kodeco.com/14214369-infinite-scrolling-pagination-in-flutter?page=1).

### Swapping List Widgets

Swap the old `ArticleListView` for your fresh `PagedArticleListView`. For that, open *lib/ui/list/article\_list\_screen.dart* and add an import to your new file at the top:

```dart
import 'package:readwenderlich/ui/list/paged_article_list_view.dart';
```

Since you’re already working with the imports, take the opportunity to do some cleaning by removing the soon-to-be unused `ArticleListView` import:

```dart
import 'package:readwenderlich/ui/list/article_list_view.dart';
```

Jump to the `build()` and replace the `Scaffold` ‘s `body` property with:

```dart
body: PagedArticleListView(
  repository: Provider.of<Repository>(context),
  listPreferences: _listPreferences,
),
```

You’re obtaining a `Repository` instance from [`Provider`](https://pub.dev/packages/provider), your dependency injection system for this project.

That’s it! You can delete the now obsolete *lib/ui/list/article\_list\_view.dart*.

Build and run. You should see your `Placeholder` in action:

[![Intermediate version of the sample project, displaying a placeholder instead of the list.](https://koenig-media.raywenderlich.com/uploads/2020/10/placeholder.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/placeholder.png)

## Engineering Infinite Scrolling Pagination

In the whole drink service situation above, you looked at infinite scrolling pagination from a product perspective. Now, put your developer glasses on, divide your goal into pieces and examine what it takes to conquer it:

[![Flow diagram of all possible listing statuses.](https://koenig-media.raywenderlich.com/uploads/2020/10/flow-diagram.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/flow-diagram.png)

[![Screenshots of every possible pagination status.](https://koenig-media.raywenderlich.com/uploads/2020/10/pagination-status-indicators.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/pagination-status-indicators.png)

- Watch the user’s scroll position so that you can fetch other pages in advance.
- Keep track of and transition between every possible status in your list.

[![Flow diagram of all possible listing statuses.](https://koenig-media.raywenderlich.com/uploads/2020/10/flow-diagram.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/flow-diagram.png)

- Keep the user posted by displaying indicators for each different status.

[![Screenshots of every possible pagination status.](https://koenig-media.raywenderlich.com/uploads/2020/10/pagination-status-indicators.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/pagination-status-indicators.png)

- Make a solution that’s reusable in different screens, possibly using other layouts. One example of this is [grids](https://flutter.dev/docs/cookbook/lists/grid-lists). Ideally, this solution should also be portable to different projects with other state management approaches.

Sounds like hard work? It doesn’t have to be. These issues are already addressed by the [Infinite Scroll Pagination](https://pub.dev/packages/infinite_scroll_pagination) package, which will be your companion for this article. In the next section, you’ll take a closer look at this package.

## Getting to Know the Package

Warm up by opening *pubspec.yaml* and replacing `# TODO: Add infinite_scroll_pagination dependency here.` with `infinite_scroll_pagination: ^3.1.0`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  infinite_scroll_pagination: ^3.1.0
```

Download your newest dependency by clicking on *Pub get* in the *Flutter commands* bar at the top of your screen.

The Infinite Scroll Pagination package makes your job as easy as stealing candy from a baby, shooting fish in a barrel or assembling a three-piece jigsaw puzzle. Speaking of the latter, here’s your first piece:

[![PagingController represented as a puzzle piece.](https://koenig-media.raywenderlich.com/uploads/2020/10/first-puzzle-piece.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/first-puzzle-piece.png)

`PagingController` is a controller for paged widgets. It’s responsible for holding the current state of the pagination and request pages from its listeners whenever needed.

If you’ve worked with Flutter’s [TextEditingController](https://api.flutter.dev/flutter/widgets/TextEditingController-class.html) or [ScrollController](https://api.flutter.dev/flutter/widgets/ScrollController-class.html), for example, you’ll feel at home with `PagingController`.

### Instantiating a PagingController

Back to *lib/ui/list/paged\_article\_list\_view.dart*, add an import to the new library at the top of the file:

```dart
import 'package:infinite_scroll_pagination/infinite_scroll_pagination.dart';
```

Now, replace `// TODO: Instantiate a PagingController.` with:

```dart
// 1
final _pagingController = PagingController<int, Article>(
  // 2
  firstPageKey: 1,
);

@override
void initState() {
  // 3
  _pagingController.addPageRequestListener((pageKey) {
    _fetchPage(pageKey);
  });
  super.initState();
}

Future<void> _fetchPage(int pageKey) async {
  // TODO: Implement the function's body.
}

@override
void dispose() {
  // 4
  _pagingController.dispose();
  super.dispose();
}
```

Here’s a step-by-step explanation of what the code above does:

1. When instantiating a `PagingController`, you need to specify two generic types. In your code, they are:
	- `int`: This is the type your endpoint uses to identify pages. For the raywenderlich.com API, that’s the page number. For [other APIs](https://nordicapis.com/everything-you-need-to-know-about-api-pagination/), instead of a page number, that could be a `String` token or the number of items to offset. Due to this diversity of pagination strategies, the package calls these identifiers *page keys*.
		- `Article`: This is the type that models your list items.
2. Remember the `int` you specified as a generic type in the previous step? Now you need to provide its initial value by using the `firstPageKey` parameter. For the raywenderlich.com API, page keys start at `1`, but other APIs might start at `0`.
3. This is how you register a callback to listen for new page requests.
4. Don’t forget to `dispose()` your controller.

### Fetching Pages

Your `_fetchPage()` implementation doesn’t have much use as it is right now. Fix this by replacing the entire function with:

```dart
Future<void> _fetchPage(int pageKey) async {
  try {
    final newPage = await widget.repository.getArticleListPage(
      number: pageKey,
      size: 8,
      // 1
      filteredPlatformIds: _listPreferences?.filteredPlatformIds,
      filteredDifficulties: _listPreferences?.filteredDifficulties,
      filteredCategoryIds: _listPreferences?.filteredCategoryIds,
      sortMethod: _listPreferences?.sortMethod,
    );

    final previouslyFetchedItemsCount =
        // 2
        _pagingController.itemList?.length ?? 0;

    final isLastPage = newPage.isLastPage(previouslyFetchedItemsCount);
    final newItems = newPage.itemList;

    if (isLastPage) {
      // 3
      _pagingController.appendLastPage(newItems);
    } else {
      final nextPageKey = pageKey + 1;
      _pagingController.appendPage(newItems, nextPageKey);
    }
  } catch (error) {
    // 4
    _pagingController.error = error;
  }
}
```

This is where all the magic happens:

1. You’re forwarding the current filtering and sorting options to the repository.
2. `itemList` is a property of `PagingController`. It holds all items loaded so far. You’re using the `?` [conditional property access](https://dart.dev/codelabs/dart-cheatsheet#conditional-property-access) because `itemList` initial value is `null`.
3. Once you have your new items, let the controller know by calling `appendPage()` or `appendLastPage()` on it.
4. If an error occurred, supply it to the controller’s `error` property.

Build and run to make sure you haven’t introduced any errors. Don’t expect any visual or functional changes.

[![Intermediate version of the sample project, displaying a placeholder instead of the list.](https://koenig-media.raywenderlich.com/uploads/2020/10/placeholder.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/placeholder.png)

## Using a Paginated ListView

Before you move on to the `build()`, there’s something you need to know:

[![PagingController and PagedListView represented as puzzle pieces.](https://koenig-media.raywenderlich.com/uploads/2020/10/second-puzzle-piece.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/second-puzzle-piece.png)

The second piece is exactly what its name suggests: a paginated version of a regular `ListView`. And as the illustration shows, it’s in there that you’ll fit in your controller.

Still on *lib/ui/list/paged\_article\_list\_view.dart*, replace the old `build()` with:

```dart
@override
Widget build(BuildContext context) =>
    // 1
    RefreshIndicator(
      onRefresh: () => Future.sync(
        // 2
        () => _pagingController.refresh(),
      ),
      // 3
      child: PagedListView.separated(
        // 4
        pagingController: _pagingController,
        padding: const EdgeInsets.all(16),
        separatorBuilder: (context, index) => const SizedBox(
          height: 16,
        ),
      ),
    );
```

Here’s what’s going on:

1. Wrapping scrollable widgets with [Flutter’s](https://api.flutter.dev/flutter/material/material-library.html) `RefreshIndicator` empowers a feature known as [*swipe to refresh*](https://material.io/design/platform-guidance/android-swipe-to-refresh.html). The user can use this to refresh the list by pulling it down from the top.
2. `PagingController` defines `refresh()`, a function for refreshing its data. You’re wrapping the `refresh()` call in a `Future`, because that’s how the `onRefresh` parameter from the `RefreshIndicator` expects it.
3. Like the good old `ListView` you already know, `PagedListView` has an alternative `separated()` constructor for adding separators between your list items.
4. You’re connecting your puzzle pieces.

### Building List Items

After all this, you suspect something might be wrong — after all, what’s going to build your list items?

The good Sherlock Holmes that you are, you investigate by hovering your magnifying glass — also known as a mouse — over `PagedListView`:

[![Android Studio warning about a missing required parameter.](https://koenig-media.raywenderlich.com/uploads/2020/10/no-builder-delegate.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/no-builder-delegate.png)

Well done, detective. You found the missing puzzle piece!

[![PagingController, PagedListView and PagedChildBuilderDelegate represented as puzzle pieces.](https://koenig-media.raywenderlich.com/uploads/2020/10/third-puzzle-piece.png)](https://koenig-media.raywenderlich.com/uploads/2020/10/third-puzzle-piece.png)

Now, it’s time to put them all together!