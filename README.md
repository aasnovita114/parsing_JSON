```
Nama        : Aas Novitasari
Nim         : 312210167
Kelas       : Ti.22.A1
Mata Kuliah : Pemograman Mobile2
```

# PARSING jSON

### Java

#### Pakage Model
- Data Item


```
  package com.umkmapps.model

import com.google.gson.annotations.SerializedName

data class DataItem(

    @field: SerializedName("id")
    val id: Int? = null,

    @field: SerializedName("email")
    val email: String? = null,


    @field: SerializedName("first_name")
    val firstName: String? = null,

    @field: SerializedName("last_name")
    val lastName: String? = null,

    @field: SerializedName("avatar")
    val avatar: String? = null,
)
```

- Response User

```
  package com.umkmapps.model

import com.google.gson.annotations.SerializedName

data class ResponseUser(

    @field: SerializedName("page")
    val page: Int? = null,

    @field: SerializedName("per_page")
    val perPage: Int? = null,

    @field: SerializedName("total")
    val total: Int? = null,

    @field: SerializedName("total_page")
    val totalPage: Int? = null,

    @field: SerializedName("data")
    val data: List<DataItem>? = null
)
```

#### Pakage Network

- Api Config
```
package com.umkmapps.network

import android.content.Context
import com.chuckerteam.chucker.api.ChuckerInterceptor
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

class ApiConfig {
    companion object{
        fun getApiService(context: Context): ApiService {
            val loggingInterceptor =
                HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY)
            val client = OkHttpClient.Builder()
                .addInterceptor(loggingInterceptor)
                .addInterceptor ( ChuckerInterceptor(context) )
                .build()
            val retrofit = Retrofit.Builder()
                .baseUrl("https://reqres.in/")
                .addConverterFactory(GsonConverterFactory.create())
                .client(client)
                .build()
            return retrofit.create(ApiService::class.java)
        }
    }
}
```

- Api Service
```
package com.umkmapps.network

import com.umkmapps.model.ResponseUser
import okhttp3.MultipartBody
import okhttp3.RequestBody
import retrofit2.Call
import retrofit2.http.Field
import retrofit2.http.FormUrlEncoded
import retrofit2.http.GET
import retrofit2.http.Multipart
import retrofit2.http.POST
import retrofit2.http.PUT
import retrofit2.http.Part
import retrofit2.http.PartMap
import retrofit2.http.Path
import retrofit2.http.Query

interface ApiService {
    @GET("api/users")
    fun getListUsers(@Query("page") page:String): Call<ResponseUser>

    //    get list user by id using path
    @GET("api/users/{id}")
    fun getUser(@Path("id") id: String): Call<ResponseUser>

    //    post user using field x-www-form-urlencoded
    @FormUrlEncoded
    @POST("Api/users")
    fun createUser(
        @Field("name") name: String,
        @Field("job") job: String
    ): Call<ResponseUser>

    //    upload file using multipart
    @Multipart
    @PUT("api/uploadfile")
    fun updateUser(
        @Part("file") file: MultipartBody.Part,
        @PartMap data: Map<String, RequestBody>
    ): Call<ResponseUser>
}
```

- MainActivity
```
package com.umkmapps

import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.umkmapps.network.ApiConfig
import com.umkmapps.model.DataItem
import com.umkmapps.model.ResponseUser
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response

class MainActivity : AppCompatActivity() {


    private lateinit var adapter: UserAdapter
    private lateinit var rv_users: RecyclerView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        rv_users = findViewById(R.id.rv_users)
        adapter = UserAdapter(mutableListOf())

        rv_users.setHasFixedSize(true)
        rv_users.layoutManager = LinearLayoutManager(this)
        rv_users.adapter = adapter

        getUser()
    }

    private fun getUser() {
        val client = ApiConfig.getApiService(this).getListUsers("1")

        client.enqueue(object : Callback<ResponseUser> {
            override fun onResponse(call: Call<ResponseUser>, response: Response<ResponseUser>) {
                if (response.isSuccessful) {
                    val dataArray = response.body()?.data as List<DataItem>
                    for (data in dataArray)
                        adapter.addUser(data)
                }
            }

            override fun onFailure(call: Call<ResponseUser>, t: Throwable) {
                Toast.makeText(this@MainActivity, t.message, Toast.LENGTH_SHORT).show()
                t.printStackTrace()
            }
        })
    }
}
```

- User Adapter
```
package com.umkmapps

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.bumptech.glide.load.resource.bitmap.CircleCrop
import com.bumptech.glide.request.RequestOptions
import com.umkmapps.model.DataItem

class UserAdapter (private val users: MutableList<DataItem>) :
    RecyclerView.Adapter<UserAdapter.ListViewHolder>(){

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ListViewHolder {
        val view: View =
            LayoutInflater.from(parent.context).inflate(R.layout.item_list_user, parent, false)
        return ListViewHolder(
            view
        )
    }

    fun addUser(newUser: DataItem){
        users.add(newUser)
        notifyItemInserted(users.lastIndex)
    }

    fun clear(){
        users.clear()
        notifyDataSetChanged()
    }

    override fun getItemCount(): Int = users.size

    override fun onBindViewHolder(holder: ListViewHolder, position: Int) {
        val user = users[position]

        Glide.with(holder.itemView.context)
            .load(user.avatar)
            .apply(RequestOptions().override(80, 80).placeholder(R.drawable.icon_avatar))
            .transform(CircleCrop())
            .into(holder.ivAvatar)

        holder.tvName.text = "${user.firstName} ${user.lastName} "
        holder.tvEmail.text = user.email
    }

    class ListViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView){
        var tvName: TextView = itemView.findViewById(R.id.itemName)
        var tvEmail: TextView = itemView.findViewById(R.id.itemEmail)
        var ivAvatar: ImageView = itemView.findViewById(R.id.itemAvatar)
    }
}
```

### Layout
- Activity_main
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

   <androidx.recyclerview.widget.RecyclerView
       android:id="@+id/rv_users"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       tools:listitem="@layout/item_list_user"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```



- Item_list_user
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/itemAvatar"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:layout_margin="16dp"
        android:contentDescription="Avatar"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:scrCompat="@tools:sample/avatars"
        />

    <TextView
        android:id="@+id/itemName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_marginLeft="16dp"
        android:textColor="#000"
        android:textSize="22sp"
        app:layout_constraintStart_toEndOf="@id/itemAvatar"
        app:layout_constraintTop_toTopOf="@id/itemAvatar"
        tools:text="Name"
        />

    <TextView
        android:id="@+id/itemEmail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:textColor="#7E7E7E"
        android:textSize="14sp"
        app:layout_constraintStart_toStartOf="@id/itemName"
        app:layout_constraintTop_toBottomOf="@id/itemName"
        tools:text="Email"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Gradle scrip
- Build_gradle.kts
```
  implementation("com.google.code.gson:gson:2.10.1")
    implementation("com.squareup.retrofit2:retrofit:2.6.4")
    implementation("com.squareup.retrofit2:converter-gson:2.6.4")
    implementation("com.squareup.okhttp3:okhttp:4.9.1")
    implementation("com.github.bumptech.glide:glide:4.16.0")
    implementation("com.github.chuckerteam.chucker:library:4.0.0")
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    implementation("com.loopj.android:android-async-http:1.4.11")
```
### Hasil
![WhatsApp Image 2024-03-22 at 15 01 13_ba276c6a](https://github.com/aasnovita114/parsing_JSON/assets/116045324/38252737-49f0-4c0f-8541-97173fac252c)

#TERIMA KASIH
