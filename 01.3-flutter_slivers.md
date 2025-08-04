# Slivers in Flutter

Slivers are a powerful concept in Flutter for creating custom scrollable areas. They allow you to create complex scrolling behaviors with multiple scrollable widgets that work together seamlessly. Slivers are the building blocks of scrollable widgets like ListView, GridView, and CustomScrollView.

## SliverAppBar

SliverAppBar is a Material Design app bar that integrates with a CustomScrollView, providing collapsing toolbar effects and flexible space.

### Basic SliverAppBar

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: CustomScrollView(
      slivers: [
        SliverAppBar(
          title: const Text('Basic SliverAppBar'),
          // Height when expanded
          expandedHeight: 200,

          // Floating: AppBar appears as soon as you scroll up
          floating: true,

          // Pinned: AppBar remains visible when collapsed
          pinned: true,

          // Snap: AppBar snaps open/closed when floating
          snap: false,

          // Stretch: Allows over-scroll stretching
          stretch: false,

          // Flexible space bar for expanded content
          flexibleSpace: FlexibleSpaceBar(
            background: Container(
              decoration: const BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                  colors: [Colors.blue, Colors.purple],
                ),
              ),
              child: const SafeArea(
                child: Center(
                  child: Text(
                    'Flexible Space',
                    style: TextStyle(fontSize: 18),
                  ),
                ),
              ),
            ),
          ),

          // Bottom widget (like TabBar)
          bottom: PreferredSize(
            preferredSize: const Size.fromHeight(48),
            child: Container(
              height: 48,
              alignment: Alignment.center,
              child: const Text(
                'Bottom Widget',
                style: TextStyle(color: Colors.white),
              ),
            ),
          ),
        ),
        SliverSafeArea(
          sliver: SliverList(
            delegate: SliverChildBuilderDelegate((context, index) {
              debugPrint('Building list item $index');
              return ListTile(
                title: Text('Item $index'),
                subtitle: Text('Subtitle for item $index'),
              );
            }, childCount: 50),
          ),
        ),
      ],
    ),
  );
}
```

### Advanced SliverAppBar with Image Background

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: CustomScrollView(
      slivers: [
        SliverAppBar(
          title: const Text('Basic SliverAppBar'),
          expandedHeight: 200,
          floating: true,
          pinned: false,
          snap: false,
          flexibleSpace: FlexibleSpaceBar(
            background: Container(
              decoration: const BoxDecoration(
                gradient: LinearGradient(
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                  colors: [Colors.blue, Colors.purple],
                ),
              ),
              child: Center(
                child: ListTile(
                  leading: CircleAvatar(
                    backgroundColor: Colors.white,
                    child: Icon(Icons.person),
                  ),
                  title: Text('Flexible Space'),
                  subtitle: Text('Subtitle for Flexible Space'),
                ),
              ),
            ),
          ),
        ),
        SliverGrid(
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 4,
            mainAxisSpacing: 10.0,
            crossAxisSpacing: 10.0,
            childAspectRatio: 4 / 3,
          ),
          delegate: SliverChildBuilderDelegate((context, index) {
            debugPrint('Building grid item $index');
            return Container(
              alignment: Alignment.center,
              color: Colors.teal[100 * (index % 9)],
              child: Text('grid item $index'),
            );
          }, childCount: 15),
        ),
        SliverList(
          delegate: SliverChildBuilderDelegate((context, index) {
            debugPrint('Building list item $index');
            return ListTile(
              title: Text('Item $index'),
              subtitle: Text('Subtitle for item $index'),
            );
          }, childCount: 50),
        ),
      ],
    ),
  );
}
```

### NestedScrollView

NestedScrollView is a widget that allows you to have a scrollable area that can be nested inside another scrollable area. For example, you have a list of items in a tab view and you want to link the scroll position with the outer scrollable area.

```dart
final List<String> tabs = <String>['Tab 1', 'Tab 2'];

@override
Widget build(BuildContext context) {
  return DefaultTabController(
    length: tabs.length,
    child: Scaffold(
      body: NestedScrollView(
        headerSliverBuilder: (context, innerBoxScrolled) {
          // These are the slivers that show up in the "outer" scroll view.
          return <Widget>[
            SliverOverlapAbsorber(
              // This widget takes the overlapping behavior of the SliverAppBar,
              // and redirects it to the SliverOverlapInjector below. If it is
              // missing, then it is possible for the nested "inner" scroll view
              // below to end up under the SliverAppBar even when the inner
              // scroll view thinks it has not been scrolled.
              // This is not necessary if the "headerSliverBuilder" only builds
              // widgets that do not overlap the next sliver.
              handle: NestedScrollView.sliverOverlapAbsorberHandleFor(
                context,
              ),
              sliver: SliverAppBar(
                floating: false,
                pinned: true,
                expandedHeight: 200,
                flexibleSpace: FlexibleSpaceBar(
                  background: Container(
                    decoration: const BoxDecoration(
                      gradient: LinearGradient(
                        begin: Alignment.topLeft,
                        end: Alignment.bottomRight,
                        colors: [Colors.blue, Colors.purple],
                      ),
                    ),
                  ),
                ),
                // The "forceElevated" property causes the SliverAppBar to show
                // a shadow. The "innerBoxScrolled" parameter is true when the
                // inner scroll view is scrolled beyond its "zero" point, i.e.
                // when it appears to be scrolled below the SliverAppBar.
                // Without this, there are cases where the shadow would appear
                // or not appear inappropriately, because the SliverAppBar is
                // not actually aware of the precise position of the inner
                // scroll views.
                forceElevated: innerBoxScrolled,
                bottom: TabBar(
                  labelColor: Colors.white,
                  unselectedLabelColor: Colors.white70,
                  indicatorColor: Colors.white70,
                  indicatorWeight: 2,
                  indicatorSize: TabBarIndicatorSize.tab,
                  indicatorPadding: EdgeInsets.symmetric(horizontal: 16),
                  // These are the widgets to put in each tab in the tab bar.
                  tabs: tabs.map((String name) => Tab(text: name)).toList(),
                ),
              ),
            ),
          ];
        },
        body: TabBarView(
          children: tabs.map((String name) {
            return SafeArea(
              top: false,
              bottom: false,
              child: Builder(
                // This Builder is needed to provide a BuildContext that is
                // "inside" the NestedScrollView, so that
                // sliverOverlapAbsorberHandleFor() can find the
                // NestedScrollView.
                builder: (BuildContext context) {
                  return CustomScrollView(
                    // The "controller" and "primary" members should be left
                    // unset, so that the NestedScrollView can control this
                    // inner scroll view.
                    // If the "controller" property is set, then this scroll
                    // view will not be associated with the NestedScrollView.
                    // The PageStorageKey should be unique to this ScrollView;
                    // it allows the list to remember its scroll position when
                    // the tab view is not on the screen.
                    key: PageStorageKey<String>(name),
                    slivers: <Widget>[
                      SliverOverlapInjector(
                        // This is the flip side of the SliverOverlapAbsorber
                        // above.
                        handle:
                            NestedScrollView.sliverOverlapAbsorberHandleFor(
                              context,
                            ),
                      ),
                      SliverPadding(
                        padding: const EdgeInsets.all(8.0),
                        // In this example, the inner scroll view has
                        // fixed-height list items, hence the use of
                        // SliverFixedExtentList. However, one could use any
                        // sliver widget here, e.g. SliverList or SliverGrid.
                        sliver: SliverFixedExtentList(
                          // The items in this example are fixed to 48 pixels
                          // high. This matches the Material Design spec for
                          // ListTile widgets.
                          itemExtent: 48.0,
                          delegate: SliverChildBuilderDelegate(
                            (BuildContext context, int index) {
                              // This builder is called for each child.
                              // In this example, we just number each list item.
                              return ListTile(title: Text('Item $index'));
                            },
                            // The childCount of the SliverChildBuilderDelegate
                            // specifies how many children this inner list
                            // has. In this example, each tab has a list of
                            // exactly 30 items, but this is arbitrary.
                            childCount: 30,
                          ),
                        ),
                      ),
                    ],
                  );
                },
              ),
            );
          }).toList(),
        ),
      ),
    ),
  );
}
```

### Common Sliver Widgets

- **SliverAppBar**: Collapsible app bar
- **SliverList**: Scrollable list of widgets
- **SliverGrid**: Scrollable grid of widgets
- **SliverToBoxAdapter**: Wraps a single box widget to work in sliver context
- **SliverPadding**: Adds padding around other slivers
- **SliverPersistentHeader**: Creates custom persistent headers
- **SliverFillViewport**: Children that fill the viewport
- **SliverFillRemaining**: Takes up remaining space in scroll view
