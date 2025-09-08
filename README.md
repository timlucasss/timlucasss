# About Me

Hi, I'm Tim, a mobile app developer passionate about creating engaging and impactful applications.  
I focus on building apps that combine fitness, fun, and social features.  

---

# Featured Project: Kompii

**Kompii** is a social fitness app that makes working out more fun and competitive.  
It is available on both platforms:

- [Get it on Google Play](https://play.google.com/store/apps/details?id=com.rtkolabs.levelup&hl=en_GB)  
- [Download on the App Store](https://apps.apple.com/us/app/kompii/id6475413819)  

Learn more at [rtkolabs.com](https://rtkolabs.com).  

---

# About This Repository

This repository does not contain the full Kompii codebase (the project is private).  
Instead, it highlights selected code snippets and technical concepts to demonstrate my approach.  

---

# Technical Highlights

## 3D Graphics with Google Filament

In Kompii, I use Google Filament to render smooth and performant 3D avatars.  
This enables real-time customization with:

- Multiple hairstyles, clothes, and accessories  
- Dynamic lighting and shadows  
- High-quality textures optimized for mobile  

Preview of the avatar system in action:  
<img src="assets/AvatarGIF.gif" width="290">

### Avatar3D Snippets

Here are a few **highlighted code snippets** from Kompii's avatar system:

#### 1. Dynamic Hair Selection
Show or hide specific hairstyles in the avatar:

```kotlin
fun showOnlyHairstyleByName(targetName: String, modelViewer: ModelViewer) {
    val asset = modelViewer.asset ?: return
    val scene = modelViewer.scene

    // Show all hair entities
    for (entity in asset.entities) {
        val entityName = asset.getName(entity).lowercase()
        if (entityName.contains("hair")) scene.addEntity(entity)
    }

    // Hide hair not matching target
    for (entity in asset.entities) {
        val entityName = asset.getName(entity).lowercase()
        if (entityName.contains("hair") && !entityName.startsWith(targetName.lowercase())) {
            scene.removeEntity(entity)
        }
    }
}
```

#### 2. Material Coloring
This function applies a color overlay to an avatar's material at runtime. It blends a base hex color with an existing texture, allowing dynamic customization of avatar appearances such as hair, clothes, or accessories.

```kotlin
private fun applyColorToMaterial(
    material: MaterialInstance,
    hexColor: String,
    textureResourceId: Int,
    blendPercentage: Float,
    overlayMode: PorterDuff.Mode
) {
    val baseColorInt = Color.parseColor(hexColor)
    val adjustedAlpha = (Color.alpha(baseColorInt) * blendPercentage).toInt()
    val blendedColor = baseColorInt and 0x00FFFFFF or (adjustedAlpha shl 24)

    val originalBitmap = BitmapFactory.decodeResource(context.resources, textureResourceId)
    val resizedBitmap = resizeBitmap(originalBitmap, 256, 256)

    val modifiedBitmap = resizedBitmap.copy(Bitmap.Config.ARGB_8888, true)
    Canvas(modifiedBitmap).drawBitmap(modifiedBitmap, 0f, 0f, Paint().apply {
        colorFilter = PorterDuffColorFilter(blendedColor, overlayMode)
    })

    val byteBuffer = ByteBuffer.allocateDirect(modifiedBitmap.width * modifiedBitmap.height * 4)
    modifiedBitmap.copyPixelsToBuffer(byteBuffer)
    byteBuffer.rewind()

    val texture = Texture.Builder()
        .width(modifiedBitmap.width)
        .height(modifiedBitmap.height)
        .levels(1)
        .sampler(Texture.Sampler.SAMPLER_2D)
        .format(Texture.InternalFormat.SRGB8_A8)
        .build(modelViewer.engine)

    texture.setImage(modelViewer.engine, 0, Texture.PixelBufferDescriptor(
        byteBuffer, Texture.Format.RGBA, Texture.Type.UBYTE, 1
    ))
    material.setParameter("baseColorMap", texture, TextureSampler())
}
```



### Fitness & Social Features Features

#### 3. Generating Challenges

<img src="assets/9.png" width="290" align="right">

Generates a pool of weekly fitness challenges, scales targets by user difficulty, and selects three unique challenges randomly each day:

```kotlin
val weeklyChallengePool = mutableListOf<Challenge>()
val weeklyTarget = (20 * 7 * (1 + profiledifficultyD!! / 10.0)).toInt() // 7 days

weeklyChallengePool.add(
    Challenge(
        title = "This Week's Activity Goal",
        text = "Complete $weeklyTarget min of fitness this week.",
        progress = "Time this week: $weeklyAllFitnessCounter min",
        isCompleted = { weeklyAllFitnessCounter >= weeklyTarget }
    )
)

if (footballEnable) {
    weeklyChallengePool.add(
        Challenge(
            title = "Football Time",
            text = "Play football for $weeklyFootballEndVal minutes this week.",
            imageName = "soccerball_inverse",
            progress = "Time this week: $profileWeeklyFootballCounter min",
            isCompleted = { profileWeeklyFootballCounter!! >= weeklyFootballEndVal }
        )
    )
}

val weeklySelectedIndices = mutableSetOf(
    weeklyRNG1!! % weeklyChallengePool.size,
    weeklyRNG2!! % weeklyChallengePool.size,
    weeklyRNG3!! % weeklyChallengePool.size
)

// Ensure 3 unique indices
var add = 1
while (weeklySelectedIndices.size < 3)
    weeklySelectedIndices.add((weeklySelectedIndices.elementAt(0) + add++) % weeklyChallengePool.size)

```



#### 4. Loading Leaderboard from Query

<img src="assets/3.png" width="290" align="right">

Fetches the user’s friends list IDs from SharedPreferences, updates each friend’s latest profile data from Firebase, and then sorts them by fitness level to display in the leaderboard.


```
fun loadLeaderboard() {
    val sharedPref = activity?.getPreferences(Context.MODE_PRIVATE) ?: return
    leaderboardRV.layoutManager = LinearLayoutManager(context)

    val friendsQuery = mFriendsDatabase.child(user.uid)
        .orderByChild("/status").equalTo("1")

    friendsData = mutableListOf()

    friendsQuery.addListenerForSingleValueEvent(object : ValueEventListener {
        override fun onDataChange(snapshot: DataSnapshot) {
            val remaining = snapshot.childrenCount.toInt()
            if (remaining == 0) return

            snapshot.children.forEach { child ->
                val friendId = child.key ?: return@forEach
                FirebaseDatabase.getInstance().getReference("Users").child(friendId)
                    .addListenerForSingleValueEvent(object : ValueEventListener {
                        override fun onDataChange(profileSnap: DataSnapshot) {
                            profileSnap.getValue<UserData>()?.let { profile ->
                                friendsData.add(Friend.fromUserData(profile))
                            }

                            // When all friends are loaded → sort + update UI
                            if (friendsData.size == remaining) {
                                val sortedFriends = friendsData.sortedByDescending { it.fitness }
                                leaderboardRV.adapter = LeaderboardAdapter(sortedFriends)

                                sharedPref.edit()
                                    .putString("${user.uid}_sortedList", Gson().toJson(sortedFriends))
                                    .apply()
                            }
                        }
                        override fun onCancelled(error: DatabaseError) {}
                    })
            }
        }
        override fun onCancelled(error: DatabaseError) {}
    })
}
```
