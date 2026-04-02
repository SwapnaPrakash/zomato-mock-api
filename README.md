# 🍔 Zomato Clone — Mock API

> GitHub Pages JSON mock API for the Online Food Ordering Android App (PIP Project)

## 🚀 Setup (One-time, 5 minutes)

### Step 1 — Create GitHub Repo
1. Go to **github.com** → New Repository
2. Name it: `zomato-mock-api`
3. Set to **Public**
4. Click **Create repository**

### Step 2 — Upload all JSON files
Upload all `.json` files from this folder to the repo root.

### Step 3 — Enable GitHub Pages
1. Go to repo **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** → folder: **/ (root)**
4. Click **Save**
5. Wait ~2 minutes → your API is live!

### Step 4 — Your Base URL
```
https://YOUR_GITHUB_USERNAME.github.io/zomato-mock-api/
```

---

## 📡 All Endpoints (Raw URLs)

Replace `YOUR_USERNAME` with your actual GitHub username.

| Screen | Endpoint | Full URL |
|--------|----------|----------|
| Home — Categories | `GET /categories.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/categories.json` |
| Home — Offers | `GET /collections.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/collections.json` |
| Home — Stores Near You | `GET /geocode.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/geocode.json` |
| Search Screen | `GET /search.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/search.json` |
| Search — Cuisines Filter | `GET /cuisines.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/cuisines.json` |
| Search — Type Filter | `GET /establishments.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/establishments.json` |
| Location Picker | `GET /locations.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/locations.json` |
| Restaurant Screen | `GET /restaurant.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/restaurant.json` |
| Restaurant — Full Menu | `GET /dailymenu.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/dailymenu.json` |
| Restaurant — Reviews | `GET /reviews.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/reviews.json` |
| Profile — User Info | `GET /user.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/user.json` |
| Profile — Order History | `GET /orders.json` | `https://YOUR_USERNAME.github.io/zomato-mock-api/orders.json` |

---

## 🔌 Retrofit Setup in Android

```kotlin
// di/NetworkModule.kt
@Provides @Singleton
fun provideRetrofit(): Retrofit = Retrofit.Builder()
    .baseUrl("https://YOUR_USERNAME.github.io/zomato-mock-api/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

```kotlin
// data/remote/api/FoodApi.kt
interface FoodApi {

    // Home Screen
    @GET("categories.json")
    suspend fun getCategories(): CategoriesResponse

    @GET("collections.json")
    suspend fun getCollections(): CollectionsResponse

    @GET("geocode.json")
    suspend fun getGeocode(): GeocodeResponse

    // Search Screen
    @GET("search.json")
    suspend fun searchRestaurants(): SearchResponse

    @GET("cuisines.json")
    suspend fun getCuisines(): CuisinesResponse

    @GET("establishments.json")
    suspend fun getEstablishments(): EstablishmentsResponse

    @GET("locations.json")
    suspend fun getLocations(): LocationSuggestionsResponse

    // Restaurant Screen
    @GET("restaurant.json")
    suspend fun getRestaurant(): RestaurantDto

    @GET("dailymenu.json")
    suspend fun getDailyMenu(): DailyMenuResponse

    @GET("reviews.json")
    suspend fun getReviews(): ReviewsResponse

    // Profile Screen
    @GET("user.json")
    suspend fun getUser(): UserResponse

    @GET("orders.json")
    suspend fun getOrders(): OrdersResponse
}
```

---

## 📦 Data Summary

| File | Data |
|------|------|
| `categories.json` | 6 categories (Delivery, Dine-out, Nightlife, Takeaway, Cafes, Catching-up) |
| `collections.json` | 5 offer banners (Trending, Newly Opened, Budget, Late Night, Pure Veg) |
| `geocode.json` | 8 restaurants near Koramangala, Bengaluru |
| `search.json` | Same 8 restaurants as search results |
| `cuisines.json` | 12 cuisines (Biryani, Pizza, Burger, North Indian, South Indian...) |
| `establishments.json` | 10 restaurant types |
| `locations.json` | 5 Bengaluru localities |
| `restaurant.json` | Full detail for Meghana Foods |
| `dailymenu.json` | 6 menu categories × 17 dishes with customisations |
| `reviews.json` | 5 realistic reviews with user profiles |
| `user.json` | Profile: Rahul Verma, 2 saved addresses |
| `orders.json` | 5 past orders with item breakdown |

---

## ⚠️ Limitation & Workaround

GitHub Pages serves **static JSON only** — no query parameters work.
So `search.json?q=pizza` returns the same data as `search.json`.

**Handle this in your Repository layer:**

```kotlin
// Filter locally in SearchRepositoryImpl
override fun searchRestaurants(query: String, filters: SearchFilters): Flow<Result<List<Restaurant>>> = flow {
    val all = api.searchRestaurants().restaurants.map(mapper::toDomain)
    val filtered = all.filter { restaurant ->
        query.isEmpty() || restaurant.name.contains(query, ignoreCase = true)
            || restaurant.categories.any { it.contains(query, ignoreCase = true) }
    }
    emit(Result.success(filtered))
}.flowOn(ioDispatcher)
```

This is **actually better for your Kotest BehaviorSpec tests** — you test the filtering logic directly!

---

## 🧪 Kotest Test Example

```kotlin
given("search query is 'biryani'") {
    `when`("searchRestaurants is called") {
        then("only biryani restaurants are returned") {
            val results = repo.searchRestaurants("biryani", SearchFilters()).first().getOrThrow()
            results.all { "biryani" in it.categories.map { c -> c.lowercase() } } shouldBe true
        }
    }
}
```
