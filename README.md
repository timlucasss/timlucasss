## Hi there 👋

I’m Tim, a **mobile app developer** passionate about creating engaging and impactful applications.  
Currently, I’m focusing on building apps that combine **fitness, fun, and social features**.

### 🚀 Featured Project
**Kompii** – a social fitness app that makes working out more fun and competitive.  
Available now on both platforms:

- [🤖 Get it on Google Play](https://play.google.com/store/apps/details?id=com.rtkolabs.levelup&hl=en_GB)  
- [📱 Download on the App Store](https://apps.apple.com/us/app/kompii/id6475413819)  

You can also learn more at my promotional website: [🌐 rtkolabs.com](https://rtkolabs.com)

---

💡 On this GitHub, I mostly share **snippets, concepts, and demos** of my work rather than full codebases.  
If you’d like to know more about my projects or code, feel free to reach out!

### 🎨 3D Graphics with Google Filament
In Kompii, I use **Google Filament** to render smooth and performant 3D avatars.  
This allows users to customize their avatar in real-time with:

- Multiple hairstyles, clothes, and accessories  
- Dynamic lighting and shadows  
- High-quality textures optimized for mobile  

Here’s a preview of the avatar system in action:  
<img src="assets/AvatarGIF.gif" width="320">

---

### 🖥️ Avatar3D Snippets

Here are a few **highlighted code snippets** from Kompii’s avatar system:

#### 1️⃣ Dynamic Hair Selection
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
