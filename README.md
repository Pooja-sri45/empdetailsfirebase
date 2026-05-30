
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: POOJASRI L
Registeration Number : 212223220076
*/
```
## ActivityMain.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <!-- USER ID -->
    <EditText
        android:id="@+id/editTextID"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="User ID"
        android:inputType="number" />

    <!-- USER NAME -->
    <EditText
        android:id="@+id/editTextUserName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="User Name"
        android:inputType="textPersonName" />

    <!-- PASSWORD -->
    <EditText
        android:id="@+id/editTextPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Password"
        android:inputType="textPassword" />

    <!-- INSERT BUTTON -->
    <Button
        android:id="@+id/btnInsert"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="INSERT"
        android:onClick="btnInsertPressed" />

    <!-- FETCH BUTTON -->
    <Button
        android:id="@+id/btnFetch"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="FETCH"
        android:onClick="btnFetchPressed" />

    <!-- UPDATE BUTTON -->
    <Button
        android:id="@+id/btnUpdate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="UPDATE"
        android:onClick="btnUpdatePressed" />

    <!-- DELETE BUTTON -->
    <Button
        android:id="@+id/btnDelete"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="DELETE"
        android:onClick="btnDeletePressed" />

</LinearLayout>
```
## MainActivity.java
```
package com.example.pmdexpt1;

import androidx.appcompat.app.AppCompatActivity;

import android.annotation.SuppressLint;
import android.database.Cursor;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    EditText editUserID, editUserName, editUserPassword;
    DatabaseManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editUserID = findViewById(R.id.editTextID);
        editUserName = findViewById(R.id.editTextUserName);
        editUserPassword = findViewById(R.id.editTextPassword);

        dbManager = new DatabaseManager(this);

        try {
            dbManager.open();
        } catch (Exception e) {
            Log.e("DB_ERROR", "Error opening database", e);
        }
    }

    // INSERT
    public void btnInsertPressed(View v) {
        String name = editUserName.getText().toString();
        String password = editUserPassword.getText().toString();

        if (name.isEmpty() || password.isEmpty()) {
            Toast.makeText(this, "Enter all fields", Toast.LENGTH_SHORT).show();
            return;
        }

        dbManager.insert(name, password);
        Toast.makeText(this, "Inserted Successfully", Toast.LENGTH_SHORT).show();
    }

    // FETCH
    public void btnFetchPressed(View v) {
        try (Cursor cursor = dbManager.fetch()) {

            if (cursor != null && cursor.moveToFirst()) {

                do {
                    @SuppressLint("Range")
                    String id = cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_ID));

                    @SuppressLint("Range")
                    String username = cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_NAME));

                    @SuppressLint("Range")
                    String password = cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_PASSWORD));

                    Log.i("DB_DATA", "ID: " + id + " Name: " + username + " Password: " + password);

                } while (cursor.moveToNext());

            } else {
                Toast.makeText(this, "No Data Found", Toast.LENGTH_SHORT).show();
            }

        } catch (Exception e) {
            Log.e("DB_ERROR", "Fetch error", e);
        }
    }

    // UPDATE
    public void btnUpdatePressed(View v) {
        try {
            long id = Long.parseLong(editUserID.getText().toString());
            String name = editUserName.getText().toString();
            String password = editUserPassword.getText().toString();

            dbManager.update(id, name, password);
            Toast.makeText(this, "Updated Successfully", Toast.LENGTH_SHORT).show();

        } catch (Exception e) {
            Log.e("DB_ERROR", "Update error", e);
            Toast.makeText(this, "Invalid ID", Toast.LENGTH_SHORT).show();
        }
    }

    // DELETE
    public void btnDeletePressed(View v) {
        try {
            long id = Long.parseLong(editUserID.getText().toString());

            dbManager.delete(id);
            Toast.makeText(this, "Deleted Successfully", Toast.LENGTH_SHORT).show();

        } catch (Exception e) {
            Log.e("DB_ERROR", "Delete error", e);
            Toast.makeText(this, "Invalid ID", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (dbManager != null) {
            dbManager.close();
        }
    }
}
```
## DatabaseManager.java
```
Package com.example.pmdexpt1;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class DatabaseManager {

    private DatabaseHelper dbHelper;
    private SQLiteDatabase database;
    private Context context;

    public DatabaseManager(Context c) {
        context = c;
    }

    public DatabaseManager open() {
        dbHelper = new DatabaseHelper(context);
        database = dbHelper.getWritableDatabase();
        return this;
    }

    public void close() {
        dbHelper.close();
    }

    // INSERT
    public void insert(String name, String password) {

        ContentValues values = new ContentValues();

        values.put(DatabaseHelper.USER_NAME, name);
        values.put(DatabaseHelper.USER_PASSWORD, password);

        database.insert(DatabaseHelper.TABLE_NAME, null, values);
    }

    // FETCH
    public Cursor fetch() {

        String[] columns = {
                DatabaseHelper.USER_ID,
                DatabaseHelper.USER_NAME,
                DatabaseHelper.USER_PASSWORD
        };

        Cursor cursor = database.query(
                DatabaseHelper.TABLE_NAME,
                columns,
                null,
                null,
                null,
                null,
                null
        );

        if (cursor != null) {
            cursor.moveToFirst();
        }

        return cursor;
    }

    // UPDATE
    public int update(long id, String name, String password) {

        ContentValues values = new ContentValues();

        values.put(DatabaseHelper.USER_NAME, name);
        values.put(DatabaseHelper.USER_PASSWORD, password);

        return database.update(
                DatabaseHelper.TABLE_NAME,
                values,
                DatabaseHelper.USER_ID + "=" + id,
                null
        );
    }

    // DELETE
    public void delete(long id) {

        database.delete(
                DatabaseHelper.TABLE_NAME,
                DatabaseHelper.USER_ID + "=" + id,
                null
        );
    }
}
```
## DatabaseHelper.java
```
package com.example.pmdexpt1;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    public static final String DATABASE_NAME = "UserDB";
    public static final int DATABASE_VERSION = 1;

    public static final String TABLE_NAME = "users";

    public static final String USER_ID = "_id";
    public static final String USER_NAME = "name";
    public static final String USER_PASSWORD = "password";

    private static final String CREATE_TABLE =
            "CREATE TABLE " + TABLE_NAME + " ("
                    + USER_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, "
                    + USER_NAME + " TEXT NOT NULL, "
                    + USER_PASSWORD + " TEXT NOT NULL);";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_TABLE);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
        onCreate(db);
    }
}



```

## OUTPUT
<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/829a9521-2eeb-4329-bc0f-7f76f81bad90" />

<img width="1918" height="1077" alt="image" src="https://github.com/user-attachments/assets/281b6b81-c717-4665-a34b-9afb65fc59da" />


## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
