---
title: Using WAPI to Build a Lore Journal
---

# Wynncraft API Lore Journal

This short guide shows how to create a simple Fabric mod for **Minecraft 1.21.4** that opens a journal menu with the `J` key. The mod uses the [Wynncraft API](https://docs.wynncraft.com/) to fetch lore unlocked by completing quests or defeating specific mobs.

## Setting up the Project

1. Use the official [Fabric example mod](https://github.com/FabricMC/fabric-example-mod) as a starting point.
2. Add WAPI as a dependency using your preferred HTTP library. The Wynncraft API base endpoint is `https://api.wynncraft.com/v3`.

```gradle
repositories {
    mavenCentral()
}

dependencies {
    // Fabric API
    modImplementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_version}"
    // Your HTTP client (e.g., OkHttp)
    implementation 'com.squareup.okhttp3:okhttp:4.12.0'
}
```

## Listening for a Key Press

Register a `KeyBinding` for the `J` key and check it every tick. When pressed, open a screen that displays the journal.

```java
public class JournalModClient implements ClientModInitializer {
    private static final KeyBinding OPEN_JOURNAL = KeyBindingHelper.registerKeyBinding(
        new KeyBinding(
            "key.journal.open",
            InputUtil.Type.KEYSYM,
            GLFW.GLFW_KEY_J,
            "key.categories.misc"
        )
    );

    @Override
    public void onInitializeClient() {
        ClientTickEvents.END_CLIENT_TICK.register(client -> {
            while (OPEN_JOURNAL.wasPressed()) {
                client.setScreen(new JournalScreen());
            }
        });
    }
}
```

## Fetching Lore Data

The lore can be queried from the Wynncraft API. For example, you may request completed quests for a player and parse their lore entries.

```java
HttpUrl url = HttpUrl.parse("https://api.wynncraft.com/v3/quest/list/%s".formatted(playerName));
Request request = new Request.Builder().url(url).build();
try (Response response = httpClient.newCall(request).execute()) {
    String body = response.body().string();
    // Parse JSON and store unlocked lore
}
```

Cache the data locally so the screen can show unlocked lore offline. Update the cache whenever the player completes a new quest or kills a required mob.

## Displaying the Journal Screen

Create a `JournalScreen` extending `Screen`. It should list all lore entries the player has unlocked. You can store them in a simple JSON file in the config directory or use the built-in `PlayerEntity` NBT data.

```
public class JournalScreen extends Screen {
    protected JournalScreen() {
        super(Text.literal("Lore Journal"));
    }
    // Render lore entries here
}
```

## Next Steps

- Check the official Wynncraft API documentation for rate limits and data formats.
- Design a better GUI using widgets from `net.minecraft.client.gui.widget`.
- Store unlocked lore in NBT or a local config file so progress persists across sessions.

