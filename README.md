# Fetch posts from an API

This code takes the raw response from the server (stream of bytes), converts it into text, reads it line by line, and builds the full JSON response string.

## AndroidManifest.xml

Permission to access the internet

``<uses-permission android:name="android.permission.INTERNET" />``

## activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Fetch Posts from API"
        android:textSize="36dp"
        android:gravity="center"
        />
    <TextView
        android:id="@+id/count"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Fetched posts: ---"
        android:textSize="24dp"
        />
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <LinearLayout
            android:id="@+id/postsLL"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"/>
    </ScrollView>
</LinearLayout>
```

## MainActivity.java

```java
package com.example.apiexample;


import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.os.StrictMode;
import android.util.Log;
import android.widget.LinearLayout;
import android.widget.TextView;

import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);

        fetchPosts();
    }

    private void fetchPosts() {
        List<Post> posts = new ArrayList<>();
        try {
                URL url = new URL("https://jsonplaceholder.typicode.com/posts");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();

                conn.setRequestMethod("GET");
                conn.setConnectTimeout(5000);
                conn.setReadTimeout(5000);

                int status = conn.getResponseCode();
                if (status == 200) {
                    BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                    StringBuilder response = new StringBuilder();
                    String line;

                    while ((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    reader.close();

                    JSONArray jsonArray = new JSONArray(response.toString());
                    Log.d("API_RESULT", "Received " + jsonArray.length() + " posts");
                    for (int i = 0; i < jsonArray.length(); i++) {
                        JSONObject obj = jsonArray.getJSONObject(i);
                        Post post = new Post(
                                obj.getInt("id"),
                                obj.getString("title"),
                                obj.getString("body")
                        );
                        posts.add(post);
                    }
                } else {
                    Log.e("HTTP_ERROR", "Response code: " + status);
                }
                conn.disconnect();
            } catch (Exception e) {
                Log.e("API_ERROR", "Failed to fetch data", e);
            }
            ((TextView)findViewById(R.id.count)).setText("Fetched Posts: "+ posts.size());
            LinearLayout postsLL = findViewById(R.id.postsLL);
            for(int i = 0; i < posts.size(); i++){
                TextView tmp = new TextView(this);
                tmp.setText((i+1) + ". " + posts.get(i).getTitle());
                tmp.setTextSize(24);
                tmp.setPadding(25,50,0, 0);
                postsLL.addView(tmp);
            }
    }
}
```

## Post.java

```java
package com.example.apiexample;

public class Post{
    int id;
    String title, body;

    public Post(int id, String title, String body){
        this.id=id;
        this.title=title;
        this.body=body;
    }

    public String getTitle(){
        return this.title:
    }
}
```

## Notes

### 1. Get the API response stream

``conn.getInputStream()``

 * When you open a connection with ``HttpURLConnection``, the server sends back data.
 * That data comes as a stream of bytes.
 * But raw bytes aren’t easy to read directly, so we need to convert them into characters/strings.
 <hr>

### 2. Convert bytes → characters → buffered reader 

``new InputStreamReader(conn.getInputStream())``

* ``InputStreamReader`` takes those raw bytes from the input stream and converts them into characters (text).

``new BufferedReader(new InputStreamReader(conn.getInputStream()))``

* ``BufferedReader`` wraps around ``InputStreamReader`` and allows us to read the data line by line efficiently instead of one character at a time.

<hr>

### 3. Prepare to store the response

``StringBuilder response = new StringBuilder();``

* Since the response is multiple lines of text (JSON), we need somewhere to accumulate it.
* ``StringBuilder`` is used instead of plain ``String`` because it’s much more efficient when repeatedly appending strings.

<hr>

### 4. Read line by line

```java
while ((line = reader.readLine()) != null) {
    response.append(line);
}
```

* ``reader.readLine()`` reads one line of text from the API response.
* The loop continues until there’s no more data (``null``).
* Each line is appended to the ``StringBuilder``.
* After the loop, ``response.toString()`` will contain the entire JSON response as a single string.

<hr>

### 5. Close the reader

``reader.close()``

* Always close streams when done.
* This frees system resources and prevents memory leaks.
