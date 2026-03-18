# Refs

1. https://github.com/stevehodgkiss/turbo-frame-permanent
2. https://katalyst.com.au/articles/turbo-cache-control-in-practice - ???
3. https://www.youtube.com/watch?v=h3zboEkzQ3Q
4. [https://www.youtube.com/watch?v=h3zboEkzQ3Q](https://www.youtube.com/watch?v=60wMhP7V1Po)

# Raw info

A wrapping `div[data-turbo-permanent]` element is used to try to make the turbo-frame element load only once on first page load.

When clicking Page 2 (after the first page load), there are 2 extra requests to user_nav.html. After clicking Back there's an additional request to user_nav.html.

[![Screenshot](https://github.com/stevehodgkiss/turbo-frame-permanent/raw/master/screenshot.png)](https://github.com/stevehodgkiss/turbo-frame-permanent/blob/master/screenshot.png)