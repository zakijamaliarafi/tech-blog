---
heroImage: '/android-offline-first-room-database.svg'
title: 'Offline-First Android Apps using Room Database'
description: 'Build robust, offline-first Android applications using the Room persistence library and the Repository pattern.'
pubDate: 'May 17 2026'
---

Users expect apps to work flawlessly even when they drop into a subway tunnel or switch to airplane mode. An "Offline-First" architecture treats local storage as the single source of truth, syncing with the network in the background. Android's **Room** library makes this pattern straightforward and type-safe.

## Why Room?

Room provides an abstraction layer over SQLite. It eliminates boilerplate code, verifies SQL queries at compile time, and integrates seamlessly with Kotlin Coroutines and `Flow` for reactive data streams.

## Defining the Database

Room requires three main components: Entities, DAOs, and the Database class.

### 1. Entities

An Entity represents a table within the SQLite database.

```kotlin
@Entity(tableName = "articles")
data class Article(
    @PrimaryKey val id: String,
    val title: String,
    val content: String,
    val isSynced: Boolean = false // Track sync state
)
```

### 2. Data Access Objects (DAOs)

DAOs contain the SQL queries. Room validates these at compile-time, preventing SQL injection and syntax errors. By returning a `Flow`, the DAO automatically emits updates whenever the underlying table changes.

```kotlin
@Dao
interface ArticleDao {
    @Query("SELECT * FROM articles")
    fun getAllArticles(): Flow<List<Article>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertArticles(articles: List<Article>)
}
```

### 3. The Database

Tie it together by defining the database holder.

```kotlin
@Database(entities = [Article::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun articleDao(): ArticleDao
}
```

## Implementing the Repository Pattern

The Repository acts as a mediator between the local database and the remote network. The UI only ever observes the local database via `Flow`.

```kotlin
class ArticleRepository(
    private val api: NetworkApi,
    private val dao: ArticleDao
) {
    // UI directly observes this. It immediately returns local data.
    val articles: Flow<List<Article>> = dao.getAllArticles()

    // Called to refresh data from network
    suspend fun refreshArticles() {
        try {
            // Fetch from network
            val networkArticles = api.fetchArticles()
            
            // Save to database. The Flow above will automatically 
            // trigger and update the UI!
            dao.insertArticles(networkArticles.map { it.toEntity() })
        } catch (e: IOException) {
            // Network failed. Ignore. The UI still has local data!
        }
    }
}
```

## Handling Writes (Mutations)

When modifying data (e.g., liking an article), you save it locally first, mark it as unsynced, and then attempt a network request.

```kotlin
suspend fun likeArticle(id: String) {
    // 1. Optimistic update: Save locally immediately
    dao.updateLikeStatus(id, liked = true, isSynced = false)

    // 2. Schedule a network sync using WorkManager
    val syncWork = OneTimeWorkRequestBuilder<SyncWorker>().build()
    WorkManager.getInstance(context).enqueue(syncWork)
}
```

By making the local database the single source of truth, you create instantaneous UIs that degrade gracefully when connectivity is lost.
