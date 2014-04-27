---
layout: post
title: "Android Asynchronous Loader"
date: 2014-03-24 23:05:43 +0800
comments: true
categories: Android
---
####Loader概述
装载器从android3.0开始引进。它使得在activity或fragment中异步加载数据变得简单。装载器具有如下特性:

* 它们对每个Activity和Fragment都有效 
* 他们提供了异步加载数据的能力
* 它们监听数据，并在数数据发生变化时传递新的结果
* 当由于配置改变而被重新创建后，它们自动重连到上一个加载器的游标，所以不必重新查询数据

#####API概述
######LoaderManager
一个抽像类，关联到一个Activity或Fragment，管理一个或多个装载器的实例。这帮助一个应用管理那些与Activity或Fragment的生命周期相关的长时间运行的的操作。最常见的方式是与一个CursorLoader一起使用，然而应用是可以随便写它们自己的装载器以加载其它类型的数据。
每个activity或fragment只有一个LoaderManager。但是一个LoaderManager可以拥有多个装载器。

######LoadManager.LoaderCallbacks
一个用于客户端与LoaderManager交互的回调接口。例如，你使用回调方法onCreateLoader()来创建一个新的装载器。

######Loader
一个执行异步数据加载的抽象类。它是加载器的基类。你可以使用典型的CursorLoader，但是你也可以实现你自己的子类。一旦装载器被激活，它们将监视它们的数据源并且在数据改变时发送新的结果。

######AsyncTaskLoader
提供一个AsyncTask来执行异步加载工作的抽象类。

######CursorLoader
AsyncTaskLoader的子类，它查询ContentResolver然后返回一个Cursor。这个类为查询cursor以标准的方式实现了装载器的协议，它的游标查询是通过AsyncTaskLoader在后台线程中执行，从而不会阻塞界面。使用这个装载器是从一个ContentProvider异步加载数据的最好方式。相比之下，通过fragment或activity的API来执行一个被管理的查询就不行了

#####使用
一个使用装载器的应用会包含如下组件：

* 一个Activity或Fragment
* 一个LoaderManager的实例
* 一个加载被ContentProvider所支持的数据的CursorLoader．或者，你可以从Loader或AsyncTaskLoader实现你自己的装载器来从其它源加载数据
* 一个LoaderManager.LoaderCallbacks的实现．这是你创建新的装载器以及管理你的已有装载器的引用的地方
* 一个显示装载器的数据的途径，例如使用一个SimpleCursorAdapter
* 一个数据源，比如当是用CursorLoader时，它将是一个ContentProvider

#####启动一个装载器
LoaderManager管理一个Activiry或Fragment中的一个或多个装载器．但每个activity或fragment只拥有一个LoaderManager．

你通常要在activity的onCreate()方法中或fragment的onActivityCreated()方法中初始化一个装载器．你可以如下创建：
```
getLoaderManager().initLoader(0,null, this);  
```
initLoader()方法有一下参数

* 一个唯一ID来标志装载器．在这个例子中，ID是0
* 可选的参数，用于装载器初始化时(本例中是null)
* 一个LoaderManager.LoaderCallbacks的实现．被LoaderManager调用以报告装载器的事件，在这个例子中，类本实现了这个接口，所以传的是它自己：this

initLoader()保证一个装载器被初始化并激活．它具有两种可能的结果：

* 如果ID所指的装载器已经存在，那么这个装载器将被重用
* 如果装载器不存在，initLoader()就触发LoaderManager.LoaderCallbacks的方法onCreateLoader()．这是你实例化并返回一个新装载器的地方

在这两种情况中，传入的LoaderManager.LoaderCallbacks的实现都与装载器绑定在一起．并且会在装载器状态变化时被调用．如果在调用这个方法时，调用者正处于启动状态，并且所请求的装载器已存在并产生了数据，那么系统会马上调用onLoadFinished()(也就是说在initLoader()还在执行时)．所以你必须为这种情况的发生做好准备.

#####重启装载器
当你使用initLoader()时，如果指定ID的装载器已经存在，则它使用这个装载器．如果不存在呢，它将创建一个新的．但是有时你却是想丢弃旧的然后开始新的数据．

要想丢弃旧数据，你应使用restartLoader()。


#####回调
LoaderManager.LoaderCallbacks是一个回调接口，它使得客户端可以与LoaderManager进行交互．

装载器，一般指的是CursorLoader，我们希望在它停止后依然保持数据．这使得应用可以在activity或fragment的 onStop() 和onStart() 之间保持数据，所以当用户回到一个应用时，它们不需等待数据加载．你使用LoaderManager.LoaderCallbacks 的方法们，在需要时创建新的装载器，并且告诉应用什么时候要停止使用装载器的数据．

LoaderManager.LoaderCallbacks 包含以下方法们:

* onCreateLoader() —跟据传入的ID，初始化并返回一个新的装载器．
* onLoadFinished() —当一个装载器完成了它的装载过程后被调用
* onLoaderReset() —当一个装载器被重置而什其数据无效时被调用



```java Loader www.shaojie.name
public static class CursorLoaderListFragment extends ListFragment
        implements OnQueryTextListener, LoaderManager.LoaderCallbacks<Cursor> {

    // This is the Adapter being used to display the list's data.
    SimpleCursorAdapter mAdapter;

    // If non-null, this is the current filter the user has provided.
    String mCurFilter;

    @Override public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // Give some text to display if there is no data.  In a real
        // application this would come from a resource.
        setEmptyText("No phone numbers");

        // We have a menu item to show in action bar.
        setHasOptionsMenu(true);

        // Create an empty adapter we will use to display the loaded data.
        mAdapter = new SimpleCursorAdapter(getActivity(),
                android.R.layout.simple_list_item_2, null,
                new String[] { Contacts.DISPLAY_NAME, Contacts.CONTACT_STATUS },
                new int[] { android.R.id.text1, android.R.id.text2 }, 0);
        setListAdapter(mAdapter);

        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getLoaderManager().initLoader(0, null, this);
    }

    @Override public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        // Place an action bar item for searching.
        MenuItem item = menu.add("Search");
        item.setIcon(android.R.drawable.ic_menu_search);
        item.setShowAsAction(MenuItem.SHOW_AS_ACTION_IF_ROOM);
        SearchView sv = new SearchView(getActivity());
        sv.setOnQueryTextListener(this);
        item.setActionView(sv);
    }

    public boolean onQueryTextChange(String newText) {
        // Called when the action bar search text has changed.  Update
        // the search filter, and restart the loader to do a new query
        // with this filter.
        mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
        getLoaderManager().restartLoader(0, null, this);
        return true;
    }

    @Override public boolean onQueryTextSubmit(String query) {
        // Don't care about this.
        return true;
    }

    @Override public void onListItemClick(ListView l, View v, int position, long id) {
        // Insert desired behavior here.
        Log.i("FragmentComplexList", "Item clicked: " + id);
    }

    // These are the Contacts rows that we will retrieve.
    static final String[] CONTACTS_SUMMARY_PROJECTION = new String[] {
        Contacts._ID,
        Contacts.DISPLAY_NAME,
        Contacts.CONTACT_STATUS,
        Contacts.CONTACT_PRESENCE,
        Contacts.PHOTO_ID,
        Contacts.LOOKUP_KEY,
    };
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        // This is called when a new Loader needs to be created.  This
        // sample only has one Loader, so we don't care about the ID.
        // First, pick the base URI to use depending on whether we are
        // currently filtering.
        Uri baseUri;
        if (mCurFilter != null) {
            baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
                    Uri.encode(mCurFilter));
        } else {
            baseUri = Contacts.CONTENT_URI;
        }

        // Now create and return a CursorLoader that will take care of
        // creating a Cursor for the data being displayed.
        String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
                + Contacts.HAS_PHONE_NUMBER + "=1) AND ("
                + Contacts.DISPLAY_NAME + " != '' ))";
        return new CursorLoader(getActivity(), baseUri,
                CONTACTS_SUMMARY_PROJECTION, select, null,
                Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
    }

    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        // Swap the new cursor in.  (The framework will take care of closing the
        // old cursor once we return.)
        mAdapter.swapCursor(data);
    }

    public void onLoaderReset(Loader<Cursor> loader) {
        // This is called when the last Cursor provided to onLoadFinished()
        // above is about to be closed.  We need to make sure we are no
        // longer using it.
        mAdapter.swapCursor(null);
    }
}
```
Loader的使用分一下几个步骤：

*  实现自己的异步Loader ,如本篇的AsyncTaskLoader或者CursorLoader或者自己继承Loader，第二步需要用到这个Loader
*  创建数据源 XXXAdapter.
*  在Activity 或者Fragments 上实现OnQueryTextListener,LoaderManager.LoaderCallbacks接口
*  在onCreateLoader里面返回Loader，在onLoadfinished里面做清除动作，在onQueryTextChange或者onQueryTextSubmit改变数据源，在onCreate里面让Loader去执行请求数据getLoaderManager().initLoader(0, null, this)





