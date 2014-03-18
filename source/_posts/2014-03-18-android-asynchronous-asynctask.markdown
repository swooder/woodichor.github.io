---
layout: post
title: "Android Asynchronous  AsyncTask"
date: 2014-03-18 00:03:02 +0800
comments: true
categories: Android
---
####AsyncTask介绍
AsyncTask是Android为我们提供的方便编写异步任务的工具类，可以代替Thread和Handler。但是AsyncTask主要还是用来处理耗时不太长的操作，如果你需要保持一个线程在后台运行很长时间，建议使用java.util.concurrent包下的Executor, ThreadpoolExecutor和FutureTask。
由于AsyncTask是一个抽象类，所以如果我们想使用它，就必须要创建一个子类去继承它。在继承时我们可以为AsyncTask类指定三个泛型参数，这三个参数的用途如下

1.  Params
在执行AsyncTask时需要传入的参数，可用于在后台任务中使用。
2. Progress
后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位。
3. Result
当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。

AsyncTask的执行分为四个步骤，每一步都对应一个回调方法，这些方法不应该由应用程序调用，开发者需要做的就是实现这些方法。

1. onPreExecute(), 该方法将在执行实际的后台操作前被UI thread调用。可以在该方法中做一些准备工作，如在界面上显示一个进度条。 
2. doInBackground(Params...), 将在onPreExecute 方法执行后马上执行，该方法运行在后台线程中。这里将主要负责执行那些很耗时的后台计算工作。可以调用 publishProgress方法来更新实时的任务进度。该方法是抽象方法，子类必须实现。 
3. onProgressUpdate(Progress...),在publishProgress方法被调用后，UI thread将调用这个方法从而在界面上展示任务的进展情况，例如通过一个进度条进行展示。 
4.  onPostExecute(Result), 在doInBackground 执行完成后，onPostExecute 方法将被UI thread调用，后台的计算结果将通过该方法传递到UI thread. 

为了正确的使用AsyncTask类，以下是几条必须遵守的准则： 

1.  Task的实例必须在UI thread中创建 
2. execute方法必须在UI thread中调用 
3. 不要手动的调用onPreExecute(), onPostExecute(Result)，doInBackground(Params...), onProgressUpdate(Progress...)这几个方法 
4. 该task只能被执行一次，否则多次调用时将会出现异常 doInBackground方法和onPostExecute的参数必须对应，这两个参数在AsyncTask声明的泛型参数列表中指定，第一个为doInBackground接受的参数，第二个为显示进度的参数，第三个为doInBackground返回和onPostExecute传入的参数。

``` java AsyncTaks http://www.shaojie.name
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }

     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
 }
```
执行Task:
```
new DownloadFilesTask().execute(url1, url2, url3);
```

