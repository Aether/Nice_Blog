---
layout: post
title: "Connect to the Internet in Android"
subtitle: ""
date: 2018-01-15
author: Aether
category: coding
tags: CODE ANDROID JAVA
finished: true
---

### Ask permissions
Android manifest.xml

     <uses-permission android:name="android.permission.INTERNET"/>
   
### Connect to the Internet

In order to get an http connection,we just call an **open connnection** on URL.Note that doesn't actually talk to the network yet. It just creates the http URL connection object.

At this point,we could set the request method,or add header fields,of change properties of the connection.

We then get an input stream from the open connection.Next we have to read the contents of this input stream.There are many ways to do this in Java,but we've chosen to do using scanner,which is used to tokenize Streams,because it's simple an relatively fast.

By setting the delimiter to \\\\A, beginning of the Stream.We force the scanner to read the entire contents of the stream into the next token stream.

It buffers the data. This means that it not only pulls the data from the network in small chunks,but because http isn't required to give us a content size,our code needs to be ready to handle buffers of different sizes.This code automatically allocates and deallocates the buffer as needed. It also handles the character encoding for us.Specifically,it translates from UTF-8 which is the default encoding for json and JavaScript to UTF-16,the format used by android.

    public static String getResponseFromHttpUrl(URL url) throws IOException {
        HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
        try {
            InputStream in = urlConnection.getInputStream();

            Scanner scanner = new Scanner(in);
            scanner.useDelimiter("\\A");

            boolean hasInput = scanner.hasNext();
            if (hasInput) {
                return scanner.next();
            } else {
                return null;
            }
        } finally {
            urlConnection.disconnect();
        }
    }
  
Then we just have to change the makeGitHubSearchQuery method to call network utils that get response from URL rather than simply displaying the URL.Store the response in a string called GithubSearchResults.What's more,cache that io exception.

    private void makeGithubSeachQuery(){
        String githubQuery = mSearchBoxEditText.getText().toString();
        URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);
        mUrlDisplayTextView.setText(githubSearchUrl.toString());
        String githubSearchResults = null;
        try {
            githubSearchResults = NetworkUtils.getResponseFromHttpUrl(githubSearchUrl);
            mSearchBoxEditText.setText(githubSearchResults);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
### Async task
Android throws an exception when you try to access the network on the main tread.We need to run the network task on a secondary execution thread.

AsyncTask allows you to run a task on a background thread while publishing results to the UI thread.The three types used by an AsyncTask are the following.

- Params 
- Progress
- Result

These three parameters correspond to three primary functions you can override in AsyncTask.

- doInBackground
- onProgressUpdate
- onPostExecute
- onPreExecute

        public class GithubQueryTask extends AsyncTask<URL, Void, String> {
        @Override
        protected String doInBackground(URL... urls) {
            URL searchUrl = urls[0];
            String githubSearchResults = null;
            try {
                githubSearchResults = NetworkUtils.getResponseFromHttpUrl(searchUrl);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return githubSearchResults;
        }

        @Override
        protected void onPostExecute(String s) {
            if (s != null && !s.equals("")) {
                mSearchResultsTextView.setText(s);
            }
        }

Start our AsyncTask,`makeGitHubSearchQuery` replace the networking code with instantiating and executing our GitHubQueryTask.

     private void makeGithubSeachQuery(){
        String githubQuery = mSearchBoxEditText.getText().toString();
        URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);
        mUrlDisplayTextView.setText(githubSearchUrl.toString());
        String githubSearchResults = null;
        new GithubQueryTask().execute(githubSearchUrl);
    }

