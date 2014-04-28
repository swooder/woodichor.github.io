---
layout: post
title: "Android MVP 模式"
date: 2014-03-24 23:06:01 +0800
comments: true
categories: Android
---
####MVP模式与MVC模式
MVP 是从经典的模式MVC演变而来，它们的基本思想有相通的地方：Controller/Presenter负责逻辑的处理，Model提供数据，View负责显示。作为一种新的模式，MVP与MVC有着一个重大的区别：在MVP中View并不直接使用Model，它们之间的通信是通过Presenter (MVC中的Controller)来进行的，所有的交互都发生在Presenter内部，而在MVC中View会从直接Model中读取数据而不是通过 Controller
![](http://img.hb.aicdn.com/c098e0b36a1be7bb989ca451e2f084d5758eb88b4e76-x3xslP_fw658)

####什么是MVC(Model View Presenter)模式
1. 为了使得视图接口可以与模型和控制器进行交互，控制器执行一些初始化事件
2. 用户通过视图（用户接口）执行一些操作
3. 控制器处理用户行为(可以用观察着模式实现)并通知模型进行更新
4. 模型引发一些事件，以便将改变发告知视图
5. 视图处理模型变更的事件，然后显示新的模型数据
6. 用户接口等待用户的进一步操作

#### Example
首先，我们先申明View的接口，有2个简单的方法

``` java viewInterface 
public inteface IResultsView {
	//set title
	public void showTitle(String title);
	
	//set result text
	public void showResults(String results);
}
```

这些方法是说明我希望View中视图的变化，这里不关心它怎么发生的

然后再定义一个Presenter的接口
``` java presenterInteface http://www.shaojie.name
public interface IResultsPresenter{
	public void onResume(Context context);
}
```
在视图里（Activity/Fragemnt）里onResume()触发的时候，调用Presenter。
```
java ResultsPreseter 
public class ResultsPresenter implements IResultsPresenter {
    private IResultsView resultsView;

    public ResultsPresenter(IResultsView resultsView) {
        this.resultsView = resultsView;
    }

    @Override
    public void onResume(Context context) {

        // Get a title
        getTitle(context);


        // Get results
        getResults(context);

    }
}
```
这里，会触发IResultView里定义的2个函数

```
public class FragmentResults extends Fragment implements IResultsView {
    private View view;
    private TextView title, content;
    private IResultsPresenter presenter;

    public FragmentResults() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        view = inflater.inflate(R.layout.fragment_results, null);

        return view;
    }

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        presenter = new ResultsPresenter(this);

        title = (TextView) view.findViewById(R.id.results_title);
        content = (TextView) view.findViewById(R.id.results_text);
    }

    @Override
    public void onResume() {
        super.onResume();

        presenter.onResume(getActivity());
    }

    @Override
    public void showTitle(String title) {

        this.title.setText(title);
    }

    @Override
    public void showResults(String results) {

        content.setText(results);
    }
```

很明显，我们就可以脱离用户接口来测试这些逻辑
```
public class TestResultsPresenter extends AndroidTestCase {
    private IResultsPresenter resultsPresenter;
    private String resultTitle;
    private String resultData;

    public TestResultsPresenter() {
        super();
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();

        IResultsView resultsView = new TestResultsView();
        resultsPresenter = new ResultsPresenter(resultsView);
    }

    @Override
    protected void tearDown() throws Exception {
        super.tearDown();
        resultsPresenter = null;
    }

    public void testResultFormat() {
        Utilities.SmsResultManager smsResultManager = Utilities.SmsResultManager.getInstance(getContext());
        smsResultManager.removeAllSmsResults();
        smsResultManager.saveResult("foo", new SmsResult("ark", "foo"));

        // Build expected Content
        String expectedContent = "ark\t\tfoo\t\tUnknown\n";

        resultsPresenter.onResume(getContext());

        assertEquals(expectedContent, resultData);
    }


    public void testResultTitle(){

        // put something
        Date nowDate = new Date();
        DateFormat df = DateFormat.getDateTimeInstance(
                DateFormat.SHORT,
                DateFormat.SHORT,
                Locale.getDefault());

        // Set date time in prefs
        Preferences.writePreferenceValue(getContext(),
                getContext().getString(R.string.pref_emergency_date_time_key), nowDate.getTime());

        String formattedDateTime = df.format(nowDate);
        String expectedTitle = String.format(getContext().getString(R.string.results_title),
                formattedDateTime);

        // Trigger onResume event in Results Presenter
        resultsPresenter.onResume(getContext());

        // Test not null
        assertNotNull(resultTitle);

        // Test as expected
        assertEquals(expectedTitle, resultTitle);
    }


    private class TestResultsView implements IResultsView {

        @Override
        public void showTitle(String title) {
            resultTitle = title;
        }

        @Override
        public void showResults(String results) {
            resultData = results;
        }
    }

```


