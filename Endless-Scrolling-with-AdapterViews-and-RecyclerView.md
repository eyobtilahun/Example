## Overview

A common application feature is to load automatically more items as the user scrolls through the items (aka infinite scroll). This is done by triggering a request for more data once the user crosses a threshold of remaining items before they've hit the end. 

The approaches for ListView and [[RecyclerView|Using-the-RecyclerView]] (the successor to ListView) are documented here.  Both are similar in code except that the [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html) in the RecyclerView needs to be passed in to provide the necessary information to implement infinite scrolling.  

In both cases, the information needed to implement the scrolling include determining the last visible item within the list and some type of threshold value to start fetching more data before the last item has been reached.  This data can be used to decide when to load more data from an external source:

<a href="http://imgur.com/6E7X1pr.png" target="_blank"><img src="http://imgur.com/6E7X1pr.png"/></a>

To provide the appearance of endless scrolling, it's important to fetch data before the user gets to the end of the list.  Adding a threshold value therefore helps anticipate the need to append more data:

<a href="http://imgur.com/NRr6dHK.png" target="_blank"><img src="http://imgur.com/NRr6dHK.png"/></a>

## Implementing with ListView 

Every `AdapterView` has support for binding to the `OnScrollListener` events which are triggered whenever a user scrolls through the collection. Using this system, we can define a basic `EndlessScrollListener` which supports most use cases by creating our own class that extends `OnScrollListener`:

```java
import android.widget.AbsListView; 

public abstract class EndlessScrollListener implements AbsListView.OnScrollListener {
	// The minimum number of items to have below your current scroll position
	// before loading more.
	private int visibleThreshold = 5;
	// The current offset index of data you have loaded
	private int currentPage = 0;
	// The total number of items in the dataset after the last load
	private int previousTotalItemCount = 0;
	// True if we are still waiting for the last set of data to load.
	private boolean loading = true;
	// Sets the starting page index
	private int startingPageIndex = 0;

	public EndlessScrollListener() {
	}

	public EndlessScrollListener(int visibleThreshold) {
		this.visibleThreshold = visibleThreshold;
	}

	public EndlessScrollListener(int visibleThreshold, int startPage) {
		this.visibleThreshold = visibleThreshold;
		this.startingPageIndex = startPage;
		this.currentPage = startPage;
	}

	// This happens many times a second during a scroll, so be wary of the code you place here.
	// We are given a few useful parameters to help us work out if we need to load some more data,
	// but first we check if we are waiting for the previous load to finish.
	@Override
	public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) 
        {
		// If the total item count is zero and the previous isn't, assume the
		// list is invalidated and should be reset back to initial state
		if (totalItemCount < previousTotalItemCount) {
			this.currentPage = this.startingPageIndex;
			this.previousTotalItemCount = totalItemCount;
			if (totalItemCount == 0) { this.loading = true; } 
		}
		// If it's still loading, we check to see if the dataset count has
		// changed, if so we conclude it has finished loading and update the current page
		// number and total item count.

		
               // Sets the  footerViewType  
                  int footerViewType = getFooterViewType();

                  RecyclerView.Adapter adapter = view.getAdapter();
                  int lastViewType = adapter.getItemViewType(adapter.getItemCount() - 1);

                  // check the lastview is footview
                  boolean isFootView = lastViewType == footerViewType;

                  if (loading && (totalItemCount > previousTotalItemCount)) {
                  if (!isFootView) {
                       loading = false;
                      previousTotalItemCount = totalItemCount;
                     }
        }
		
		// If it isn't currently loading, we check to see if we have breached
		// the visibleThreshold and need to reload more data.
		// If we do need to reload some more data, we execute onLoadMore to fetch the data.
		if (!loading && (firstVisibleItem + visibleItemCount + visibleThreshold) >= totalItemCount ) {
		    loading = onLoadMore(currentPage + 1, totalItemCount);
		}
	}

	// Defines the process for actually loading more data based on page
	// Returns true if more data is being loaded; returns false if there is no more data to load.
	public abstract boolean onLoadMore(int page, int totalItemsCount);

        // set FooterView type
        // if don't use footview loadmore  set: -1
            public abstract int getFooterViewType();

	@Override
	public void onScrollStateChanged(AbsListView view, int scrollState) {
		// Don't take any action on changed
	}
}
```

Notice that this is an abstract class, and that in order to use this, you must extend this base class and define the `onLoadMore` method to actually retrieve the new data. We can define now an anonymous class within any activity that extends `EndlessScrollListener` and bind that to the AdapterView. For example:

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ... the usual 
        ListView lvItems = (ListView) findViewById(R.id.lvItems);
        // Attach the listener to the AdapterView onCreate
        lvItems.setOnScrollListener(new EndlessScrollListener() {
          @Override
          public boolean onLoadMore(int page, int totalItemsCount) {
              // Triggered only when new data needs to be appended to the list
              // Add whatever code is needed to append new items to your AdapterView
              customLoadMoreDataFromApi(page); 
              // or customLoadMoreDataFromApi(totalItemsCount); 
              return true; // ONLY if more data is actually being loaded; false otherwise.
          }
        });
    }
    
    // Append more data into the adapter
    public void customLoadMoreDataFromApi(int offset) {
      // This method probably sends out a network request and appends new data items to your adapter. 
      // Use the offset value and add it as a parameter to your API request to retrieve paginated data.
      // Deserialize API response and then construct new objects to append to the adapter
    }
}
```

Now as you scroll, items will be automatically filling in because the `onLoadMore` method will be triggered once the user crosses the `visibleThreshold`. This approach works equally well for a `GridView` and the listener gives access to both the `page` as well as the `totalItemsCount` to support both pagination and offset based fetching.

## Implementing with RecyclerView

We can also use a similar approach with the [[RecyclerView|Using-the-RecyclerView]] by defining an interface `EndlessRecyclerViewScrollListener` that requires an `onLoadMore()` method to be implemented.  The [LayoutManager](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html), which is responsible in the RecyclerView for rendering where items should be positioned and manages scrolling, provides information about the current scroll position relative to the adapter.  For this reason, we need to pass an instance of what LayoutManager is being used to collect the necessary information to ascertain when to load more data.  

Start by referring to this [code sample for usage](https://gist.github.com/rogerhu/17aca6ad4dbdb3fa5892) and [this code sample](https://gist.github.com/nesquena/d09dc68ff07e845cc622) for the full listener source code. In short endless pagination requires the following steps:

1. Copy over the [EndlessRecyclerViewScrollListener.java](https://gist.github.com/nesquena/d09dc68ff07e845cc622) into your application.
2. Call `addOnScrollListener(...)` on a `RecyclerView` to enable endless pagination. Pass in an instance of `EndlessRecyclerViewScrollListener` and implement the `onLoadMore` which fires whenever a new page needs to be loaded to fill up the list.
3. Inside the aforementioned `onLoadMore` method, load additional items into the adapter either by sending out a network request or by loading from another source. 

To start handling the scroll events for steps 2 and 3, we need to use the `addOnScrollListener()` method in our `Activity` or `Fragment` and pass in the instance of the `EndlessRecyclerViewScrollListener` with the layout manager as shown below:

```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       // Configure the RecyclerView
       RecyclerView rvItems = (RecyclerView) findViewById(R.id.rvContacts);
       LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
       recyclerView.setLayoutManager(linearLayoutManager);
       // Add the scroll listener
       rvItems.addOnScrollListener(new EndlessRecyclerViewScrollListener(linearLayoutManager) {
           @Override
           public void onLoadMore(int page, int totalItemsCount) {
               // Triggered only when new data needs to be appended to the list
               // Add whatever code is needed to append new items to the bottom of the list
               customLoadMoreDataFromApi(page); 
           }
      });
  }

  // Append more data into the adapter
  // This method probably sends out a network request and appends new data items to your adapter. 
  public void customLoadMoreDataFromApi(int offset) {
      // Send an API request to retrieve appropriate data using the offset value as a parameter.
      // Deserialize API response and then construct new objects to append to the adapter
      // Add the new objects to the data source for the adapter
      items.addAll(moreItems);
      // For efficiency purposes, notify the adapter of only the elements that got changed
      // curSize will equal to the index of the first element inserted because the list is 0-indexed
      int curSize = adapter.getItemCount(); 
      adapter.notifyItemRangeInserted(curSize, items.size() - 1);
  }
}
```

All of the code needed is already incorporated in the `EndlessRecyclerViewScrollListener.java` code snippet above. However, if you wish to understand how the endless scrolling is calculated, the explanation is included in the sections below.

### Using with LinearLayoutManager

The logic for RecyclerView for `LinearLayoutManager` is implemented basically the same as what is used for the `ListView`. One slight difference from the `ListView` is that that the `RecyclerView` layout managers include the position of the last visible item on the screen, so we can use this info directly instead of needing to calculate it:

```java
// This happens many times a second during a scroll, so be wary of the code you place here.
// We are given a few useful parameters to help us work out if we need to load some more data,
// but first we check if we are waiting for the previous load to finish.
@Override
public void onScrolled(RecyclerView view, int dx, int dy) {
   int lastVisibleItemPosition = mLinearLayoutManager.findLastVisibleItemPosition();
   // ...
```

This position can the be used to calculate when new items need to be loaded by comparing the last visible position and the total number of items.

### Using with StaggeredGridLayoutManager

Because the `StaggeredGridLayoutManager` enables elements to be placed in different columns, determining whether more items need to be loaded must be calculated by looking at the last visible positions across each row.  We can implement endless scrolling by determining the highest value in the row to determine whether more items need to be fetched.  Note also that the threshold value has to be increased since more items can be displayed on the screen, so we use a multiplier by calling [`getSpanCount()`](http://developer.android.com/reference/android/support/v7/widget/StaggeredGridLayoutManager.html#getSpanCount()) on the layout manager:

```java
public abstract class EndlessRecyclerViewScrollListener extends RecyclerView.OnScrollListener {
    StaggeredGridLayoutManager mStaggeredGridLayoutManager;

    public EndlessRecyclerViewScrollListener(StaggeredGridLayoutManager layoutManager) {
        this.mStaggeredGridLayoutManager = layoutManager;
        visibleThreshold = visibleThreshold * mStaggeredGridLayoutManager.getSpanCount();
    }

    public int getLastVisibleItem(int[] lastVisibleItemPositions) {
        int maxSize = 0;
        for (int i = 0; i < lastVisibleItemPositions.length; i++) {
            if (i == 0) {
                maxSize = lastVisibleItemPositions[i];
            }
            else if (lastVisibleItemPositions[i] > maxSize) {
                maxSize = lastVisibleItemPositions[i];
            }
        }
        return maxSize;
    }

    // This happens many times a second during a scroll, so be wary of the code you place here.
    // We are given a few useful parameters to help us work out if we need to load some more data,
    // but first we check if we are waiting for the previous load to finish.
    @Override
    public void onScrolled(RecyclerView view, int dx, int dy) {
        int[] lastVisibleItemPositions = mStaggeredGridLayoutManager.findLastVisibleItemPositions(null);
        lastVisibleItemPosition = getLastVisibleItem(lastVisibleItemPositions);
        int visibleItemCount = view.getChildCount();
        int totalItemCount = mStaggeredGridLayoutManager.getItemCount();
        // same code as before
    }
}
```

### Supporting Arbitrary Layout Managers

The code used for the endless scrolling implements checks to determine which layout manager is being used:

```java
public EndlessRecyclerViewScrollListener(LinearLayoutManager layoutManager) {
    this.mLayoutManager = layoutManager;
}

public EndlessRecyclerViewScrollListener(GridLayoutManager layoutManager) {
    this.mLayoutManager = layoutManager;
    // Increase visible threshold based on number of columns
    visibleThreshold = visibleThreshold * layoutManager.getSpanCount();
}

public EndlessRecyclerViewScrollListener(StaggeredGridLayoutManager layoutManager) {
    this.mLayoutManager = layoutManager;
    // Increase visible threshold based on number of columns
    visibleThreshold = visibleThreshold * layoutManager.getSpanCount();
}
```

We can then calculate the last visible position depending on the layout manager:

```java
@Override
public void onScrolled(RecyclerView view, int dx, int dy) {
  int lastVisibleItemPosition = 0;
  int totalItemCount = mLayoutManager.getItemCount();

  // Check the layout manager type in order to determine the last visible position
  if (mLayoutManager instanceof StaggeredGridLayoutManager) {
     int[] lastVisibleItemPositions = ((StaggeredGridLayoutManager) mLayoutManager).findLastVisibleItemPositions(null);
     // get maximum element within the list
     lastVisibleItemPosition = getLastVisibleItem(lastVisibleItemPositions);
  } else if (mLayoutManager instanceof LinearLayoutManager) {
     lastVisibleItemPosition = ((LinearLayoutManager) mLayoutManager).findLastVisibleItemPosition();
  } else if (mLayoutManager instanceof GridLayoutManager) {
     lastVisibleItemPosition = ((GridLayoutManager) mLayoutManager).findLastVisibleItemPosition();
  }

  // Remaining scroll logic is here
}
```

### Alternative Methods

Instead of using [EndlessRecyclerViewScrollListener.java](https://gist.github.com/nesquena/d09dc68ff07e845cc622) introduced in the previous section, we can use a simpler and less computational way to implement endless scrolling. We can make use of the following code snippet to achieve endless scrolling:

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
  EndlessScrollListener endlessScrollListener
  
  ...
  public void setEndlessScrollListener(EndlessScrollListener endlessScrollListener) {
      this.endlessScrollListener = endlessScrollListener;
  }
  
  @Override
  public void onBindViewHolder(MyAdapter.ViewHolder holder, int position) {
      final Data data = dataset.get(position);
      static final int VISIBLE_THRESHOLD = 5;

      // you can cache getItemCount() in a member variable for more performance tuning
      if(position == getItemCount() - VISIBLE_THRESHOLD) {  
          if(endlessScrollListener != null) {
              endlessScrollListener.onLoadMore(position);
          }
      }

      ...
  }
  
  @Override
  public int getItemCount() {
      if(dataset == null)
          return 0;
      else
          return dataset.size();
  }
  ...
  
  public interface EndlessScrollListener {
      /**
       * Loads more data.
       * @param position
       * @return true loads data actually, false otherwise.
       */
      boolean onLoadMore(int position);
  }
}

```

## Troubleshooting

If you are running into problems, please carefully consider the following suggestions:

* For the ListView, make sure to setup the `setOnScrollListener` listener in the `onCreate` method of the `Activity` or `onCreateView` in a Fragment and not much later otherwise you may encounter unexpected issues. 

* In order for the pagination system to continue working reliably, you should make sure to **clear the adapter** of items (or notify adapter after clearing the array) before appending new items to the list.  For RecyclerView, it is highly recommended to make more granular updates when notifying the adapter.  See this [video talk] (https://youtu.be/imsr8NrIAMs?t=8m27s) for more context.

* In order for this pagination system to trigger, keep in mind that as `customLoadMoreDataFromApi` is called, new data needs to be **appended to the existing data source**. In other words, only clear items from the list when on the initial "page". Subsequent "pages" of data should be appended to the existing data.

* For the `RecyclerView`, if you intend to clear the contents of the list and start endless scrolling again, make sure to clear the contents of the list and notify the adapter the contents have changed **as soon as possible**.  The reason is that `RecyclerView` will trigger a new `onScroll` event and allow the endless scrolling code to reset itself.   

## Displaying Progress with Custom Adapter

To display the last row as a ProgressBar indicating that the ListView is loading data, we do the trick in the Adapter. Having defined two types of views in `getItemViewType(int position)`, we can display the last row differently from a normal data row. It can be a ProgressBar or some text to indicate that the ListView has reached the last row by comparing the size of data List to the number of items on the server side. See [this gist](https://gist.github.com/nesquena/a988aac278cff59a9a69) for sample code.

## References

* <http://benjii.me/2010/08/endless-scrolling-listview-in-android/>
* <http://stackoverflow.com/questions/1080811/android-endless-list>
* <http://stackoverflow.com/questions/12583419/android-listview-automatically-load-more-data-when-scroll-to-the-bottom>