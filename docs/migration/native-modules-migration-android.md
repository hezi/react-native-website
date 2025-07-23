---
id: native-modules-migration-android
title: Native Modules Migration - Android
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Native Modules Migration - Android

This guide will walk you through migrating your existing Android native modules from the legacy architecture to the new architecture (Turbo Modules). We'll use a `CalendarModule` to demonstrate the migration process.

## Introduction

The new architecture introduces Turbo Modules, which offer improved performance through lazy loading, better type safety with code generation, and direct communication between JavaScript and native code. This guide shows you how to migrate your existing native modules step by step.

## Prerequisites

Before starting the migration, ensure you have:

1. A React Native app with the new architecture enabled
2. Basic knowledge of Android development (Java or Kotlin)
3. An existing native module that you want to migrate
4. TypeScript or Flow set up in your project

## Project Structure Overview

Here's how the structure changes between legacy and new architecture:

### Legacy Architecture
```
android/app/src/main/java/com/yourapp/
├── MainApplication.java
└── modules/
    └── CalendarModule.java
```

### New Architecture
```
android/app/src/main/java/com/yourapp/
├── MainApplication.java
└── modules/
    └── CalendarModule.java

js/
└── NativeCalendarModule.ts  # TypeScript specification
```

## Step 1: Creating the TypeScript Specification

The first step is to create a TypeScript specification that defines the interface of your native module. This specification is used to generate native code interfaces.

<Tabs groupId="js-language">
<TabItem value="ts" label="TypeScript">

```typescript title="js/NativeCalendarModule.ts"
import type {TurboModule} from 'react-native';
import {TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  // Synchronous methods
  getDefaultCalendarName(): string;
  
  // Asynchronous methods with callbacks
  getCalendarEvents(
    startDate: string,
    endDate: string,
    callback: (error: string | null, events?: Array<{
      id: string;
      title: string;
      startDate: string;
      endDate: string;
    }>) => void,
  ): void;
  
  // Promise-based methods
  createCalendarEvent(
    title: string,
    startDate: string,
    endDate: string,
    location?: string,
  ): Promise<string>;
  
  // Methods that return objects
  getCalendarById(calendarId: string): {
    id: string;
    title: string;
    isPrimary: boolean;
  } | null;
  
  // Constants (defined separately)
  getConstants(): {
    DEFAULT_EVENT_STATUS: string;
    CALENDAR_PERMISSIONS: {
      READ: string;
      WRITE: string;
    };
  };
  
  // Event emitter methods
  addListener(eventName: string): void;
  removeListeners(count: number): void;
}

export default TurboModuleRegistry.getEnforcing<Spec>('CalendarModule');
```

</TabItem>
<TabItem value="flow" label="Flow">

```javascript title="js/NativeCalendarModule.js"
// @flow

import type {TurboModule} from 'react-native';
import {TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  // Synchronous methods
  +getDefaultCalendarName: () => string;
  
  // Asynchronous methods with callbacks
  +getCalendarEvents: (
    startDate: string,
    endDate: string,
    callback: (error: ?string, events?: Array<{|
      id: string,
      title: string,
      startDate: string,
      endDate: string,
    |}>) => void,
  ) => void;
  
  // Promise-based methods
  +createCalendarEvent: (
    title: string,
    startDate: string,
    endDate: string,
    location?: string,
  ) => Promise<string>;
  
  // Methods that return objects
  +getCalendarById: (calendarId: string) => ?{|
    id: string,
    title: string,
    isPrimary: boolean,
  |};
  
  // Constants (defined separately)
  +getConstants: () => {|
    DEFAULT_EVENT_STATUS: string,
    CALENDAR_PERMISSIONS: {|
      READ: string,
      WRITE: string,
    |},
  |};
  
  // Event emitter methods
  +addListener: (eventName: string) => void;
  +removeListeners: (count: number) => void;
}

export default (TurboModuleRegistry.getEnforcing<Spec>('CalendarModule'): Spec);
```

</TabItem>
</Tabs>

## Step 2: Basic Module Setup and Registration

Now let's look at how the basic module setup changes between architectures.

### Legacy Architecture

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
package com.yourapp.modules;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableArray;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.facebook.react.module.annotations.ReactModule;

import java.util.Map;
import java.util.HashMap;

@ReactModule(name = CalendarModule.NAME)
public class CalendarModule extends ReactContextBaseJavaModule {
    public static final String NAME = "CalendarModule";
    
    public CalendarModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    
    @Override
    public String getName() {
        return NAME;
    }
    
    // Module implementation continues...
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
package com.yourapp.modules

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.ReactContextBaseJavaModule
import com.facebook.react.bridge.ReactMethod
import com.facebook.react.bridge.Callback
import com.facebook.react.bridge.Promise
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.WritableArray
import com.facebook.react.bridge.Arguments
import com.facebook.react.modules.core.DeviceEventManagerModule
import com.facebook.react.module.annotations.ReactModule

@ReactModule(name = CalendarModule.NAME)
class CalendarModule(reactContext: ReactApplicationContext) : 
    ReactContextBaseJavaModule(reactContext) {
    
    companion object {
        const val NAME = "CalendarModule"
    }
    
    override fun getName(): String = NAME
    
    // Module implementation continues...
}
```

</TabItem>
</Tabs>

### New Architecture

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
package com.yourapp.modules;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableArray;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.facebook.react.turbomodule.core.interfaces.CallbackWrapper;

import com.yourapp.NativeCalendarModuleSpec;

import java.util.Map;
import java.util.HashMap;

@ReactModule(name = CalendarModule.NAME)
public class CalendarModule extends NativeCalendarModuleSpec {
    public static final String NAME = "CalendarModule";
    
    public CalendarModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    
    @Override
    @NonNull
    public String getName() {
        return NAME;
    }
    
    // Module implementation continues...
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
package com.yourapp.modules

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.WritableArray
import com.facebook.react.bridge.Arguments
import com.facebook.react.module.annotations.ReactModule
import com.facebook.react.modules.core.DeviceEventManagerModule
import com.facebook.react.turbomodule.core.interfaces.CallbackWrapper

import com.yourapp.NativeCalendarModuleSpec

@ReactModule(name = CalendarModule.NAME)
class CalendarModule(reactContext: ReactApplicationContext) : 
    NativeCalendarModuleSpec(reactContext) {
    
    companion object {
        const val NAME = "CalendarModule"
    }
    
    override fun getName(): String = NAME
    
    // Module implementation continues...
}
```

</TabItem>
</Tabs>

:::info Key Changes
- **Base Class**: Changed from `ReactContextBaseJavaModule` to generated `NativeCalendarModuleSpec`
- **Imports**: Added imports for the generated spec class
- **Type Safety**: The generated spec provides compile-time type checking
:::

## Step 3: Migrating Synchronous Methods

Synchronous methods are the simplest to migrate. Here's how they change:

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@ReactMethod(isBlockingSynchronousMethod = true)
public String getDefaultCalendarName() {
    // Get the default calendar name
    return "Personal Calendar";
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
@ReactMethod(isBlockingSynchronousMethod = true)
fun getDefaultCalendarName(): String {
    // Get the default calendar name
    return "Personal Calendar"
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@Override
public String getDefaultCalendarName() {
    // Get the default calendar name
    return "Personal Calendar";
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
override fun getDefaultCalendarName(): String {
    // Get the default calendar name
    return "Personal Calendar"
}
```

</TabItem>
</Tabs>

### JavaScript Usage

```javascript
import NativeCalendarModule from './NativeCalendarModule';

const calendarName = NativeCalendarModule.getDefaultCalendarName();
console.log('Default calendar:', calendarName);
```

:::info Key Changes
- **No @ReactMethod annotation**: Methods are defined in the TypeScript spec
- **Override keyword**: Methods override the generated interface
- **Return types must match**: The spec ensures type safety
:::

## Step 4: Migrating Callbacks and Promises

### Callback-based Methods

#### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@ReactMethod
public void getCalendarEvents(String startDate, String endDate, Callback callback) {
    try {
        WritableArray events = Arguments.createArray();
        
        // Simulate fetching calendar events
        WritableMap event1 = Arguments.createMap();
        event1.putString("id", "1");
        event1.putString("title", "Team Meeting");
        event1.putString("startDate", startDate);
        event1.putString("endDate", endDate);
        events.pushMap(event1);
        
        WritableMap event2 = Arguments.createMap();
        event2.putString("id", "2");
        event2.putString("title", "Project Review");
        event2.putString("startDate", startDate);
        event2.putString("endDate", endDate);
        events.pushMap(event2);
        
        callback.invoke(null, events);
    } catch (Exception e) {
        callback.invoke(e.getMessage(), null);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
@ReactMethod
fun getCalendarEvents(startDate: String, endDate: String, callback: Callback) {
    try {
        val events = Arguments.createArray()
        
        // Simulate fetching calendar events
        val event1 = Arguments.createMap().apply {
            putString("id", "1")
            putString("title", "Team Meeting")
            putString("startDate", startDate)
            putString("endDate", endDate)
        }
        events.pushMap(event1)
        
        val event2 = Arguments.createMap().apply {
            putString("id", "2")
            putString("title", "Project Review")
            putString("startDate", startDate)
            putString("endDate", endDate)
        }
        events.pushMap(event2)
        
        callback.invoke(null, events)
    } catch (e: Exception) {
        callback.invoke(e.message, null)
    }
}
```

</TabItem>
</Tabs>

#### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@Override
public void getCalendarEvents(String startDate, String endDate, CallbackWrapper callback) {
    try {
        WritableArray events = Arguments.createArray();
        
        // Simulate fetching calendar events
        WritableMap event1 = Arguments.createMap();
        event1.putString("id", "1");
        event1.putString("title", "Team Meeting");
        event1.putString("startDate", startDate);
        event1.putString("endDate", endDate);
        events.pushMap(event1);
        
        WritableMap event2 = Arguments.createMap();
        event2.putString("id", "2");
        event2.putString("title", "Project Review");
        event2.putString("startDate", startDate);
        event2.putString("endDate", endDate);
        events.pushMap(event2);
        
        callback.invoke(null, events);
    } catch (Exception e) {
        callback.invoke(e.getMessage(), null);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
override fun getCalendarEvents(startDate: String, endDate: String, callback: CallbackWrapper) {
    try {
        val events = Arguments.createArray()
        
        // Simulate fetching calendar events
        val event1 = Arguments.createMap().apply {
            putString("id", "1")
            putString("title", "Team Meeting")
            putString("startDate", startDate)
            putString("endDate", endDate)
        }
        events.pushMap(event1)
        
        val event2 = Arguments.createMap().apply {
            putString("id", "2")
            putString("title", "Project Review")
            putString("startDate", startDate)
            putString("endDate", endDate)
        }
        events.pushMap(event2)
        
        callback.invoke(null, events)
    } catch (e: Exception) {
        callback.invoke(e.message, null)
    }
}
```

</TabItem>
</Tabs>

### Promise-based Methods

#### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@ReactMethod
public void createCalendarEvent(
    String title,
    String startDate,
    String endDate,
    @Nullable String location,
    Promise promise
) {
    try {
        // Simulate creating a calendar event
        String eventId = "event_" + System.currentTimeMillis();
        
        // In a real implementation, you would create the event here
        // For this example, we'll just return the generated ID
        
        promise.resolve(eventId);
    } catch (Exception e) {
        promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
@ReactMethod
fun createCalendarEvent(
    title: String,
    startDate: String,
    endDate: String,
    location: String?,
    promise: Promise
) {
    try {
        // Simulate creating a calendar event
        val eventId = "event_${System.currentTimeMillis()}"
        
        // In a real implementation, you would create the event here
        // For this example, we'll just return the generated ID
        
        promise.resolve(eventId)
    } catch (e: Exception) {
        promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e)
    }
}
```

</TabItem>
</Tabs>

#### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@Override
public void createCalendarEvent(
    String title,
    String startDate,
    String endDate,
    @Nullable String location,
    Promise promise
) {
    try {
        // Simulate creating a calendar event
        String eventId = "event_" + System.currentTimeMillis();
        
        // In a real implementation, you would create the event here
        // For this example, we'll just return the generated ID
        
        promise.resolve(eventId);
    } catch (Exception e) {
        promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
override fun createCalendarEvent(
    title: String,
    startDate: String,
    endDate: String,
    location: String?,
    promise: Promise
) {
    try {
        // Simulate creating a calendar event
        val eventId = "event_${System.currentTimeMillis()}"
        
        // In a real implementation, you would create the event here
        // For this example, we'll just return the generated ID
        
        promise.resolve(eventId)
    } catch (e: Exception) {
        promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e)
    }
}
```

</TabItem>
</Tabs>

### JavaScript Usage

```javascript
import NativeCalendarModule from './NativeCalendarModule';

// Using callbacks
NativeCalendarModule.getCalendarEvents(
  '2024-01-01',
  '2024-01-31',
  (error, events) => {
    if (error) {
      console.error('Failed to get events:', error);
    } else {
      console.log('Calendar events:', events);
    }
  }
);

// Using promises
async function createEvent() {
  try {
    const eventId = await NativeCalendarModule.createCalendarEvent(
      'Team Standup',
      '2024-01-15T10:00:00Z',
      '2024-01-15T10:30:00Z',
      'Conference Room A'
    );
    console.log('Created event with ID:', eventId);
  } catch (error) {
    console.error('Failed to create event:', error);
  }
}
```

:::info Key Changes
- **Callback type**: Changed from `Callback` to `CallbackWrapper` in new architecture
- **Promise handling**: Remains the same, but type safety is enforced by the spec
:::

## Step 5: Migrating Constants

Constants are handled differently in the new architecture.

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@Override
public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put("DEFAULT_EVENT_STATUS", "CONFIRMED");
    
    final Map<String, Object> permissions = new HashMap<>();
    permissions.put("READ", "android.permission.READ_CALENDAR");
    permissions.put("WRITE", "android.permission.WRITE_CALENDAR");
    constants.put("CALENDAR_PERMISSIONS", permissions);
    
    return constants;
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
override fun getConstants(): Map<String, Any> {
    return mapOf(
        "DEFAULT_EVENT_STATUS" to "CONFIRMED",
        "CALENDAR_PERMISSIONS" to mapOf(
            "READ" to "android.permission.READ_CALENDAR",
            "WRITE" to "android.permission.WRITE_CALENDAR"
        )
    )
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
@Override
protected Map<String, Object> getTypedExportedConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put("DEFAULT_EVENT_STATUS", "CONFIRMED");
    
    final Map<String, Object> permissions = new HashMap<>();
    permissions.put("READ", "android.permission.READ_CALENDAR");
    permissions.put("WRITE", "android.permission.WRITE_CALENDAR");
    constants.put("CALENDAR_PERMISSIONS", permissions);
    
    return constants;
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
override fun getTypedExportedConstants(): Map<String, Any> {
    return mapOf(
        "DEFAULT_EVENT_STATUS" to "CONFIRMED",
        "CALENDAR_PERMISSIONS" to mapOf(
            "READ" to "android.permission.READ_CALENDAR",
            "WRITE" to "android.permission.WRITE_CALENDAR"
        )
    )
}
```

</TabItem>
</Tabs>

### JavaScript Usage

```javascript
import NativeCalendarModule from './NativeCalendarModule';

// Access constants
const { DEFAULT_EVENT_STATUS, CALENDAR_PERMISSIONS } = NativeCalendarModule.getConstants();

console.log('Default status:', DEFAULT_EVENT_STATUS);
console.log('Read permission:', CALENDAR_PERMISSIONS.READ);
console.log('Write permission:', CALENDAR_PERMISSIONS.WRITE);
```

:::info Key Changes
- **Method name**: Changed from `getConstants()` to `getTypedExportedConstants()`
- **Type safety**: The TypeScript spec ensures constants match the expected types
:::

## Step 6: Migrating Event Emitters

Event emission requires implementing event listener methods.

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
private void sendEvent(String eventName, WritableMap params) {
    getReactApplicationContext()
        .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
        .emit(eventName, params);
}

@ReactMethod
public void startCalendarEventReminders() {
    // Simulate sending reminders
    WritableMap params = Arguments.createMap();
    params.putString("eventId", "event_123");
    params.putString("title", "Team Meeting starting in 15 minutes");
    params.putDouble("timestamp", System.currentTimeMillis());
    
    sendEvent("calendarEventReminder", params);
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
private fun sendEvent(eventName: String, params: WritableMap) {
    reactApplicationContext
        .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
        .emit(eventName, params)
}

@ReactMethod
fun startCalendarEventReminders() {
    // Simulate sending reminders
    val params = Arguments.createMap().apply {
        putString("eventId", "event_123")
        putString("title", "Team Meeting starting in 15 minutes")
        putDouble("timestamp", System.currentTimeMillis().toDouble())
    }
    
    sendEvent("calendarEventReminder", params)
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
private int listenerCount = 0;

private void sendEvent(String eventName, WritableMap params) {
    if (listenerCount > 0) {
        getReactApplicationContext()
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
            .emit(eventName, params);
    }
}

@Override
public void addListener(String eventName) {
    listenerCount += 1;
}

@Override
public void removeListeners(double count) {
    listenerCount -= (int) count;
    if (listenerCount < 0) {
        listenerCount = 0;
    }
}

public void startCalendarEventReminders() {
    // Simulate sending reminders
    WritableMap params = Arguments.createMap();
    params.putString("eventId", "event_123");
    params.putString("title", "Team Meeting starting in 15 minutes");
    params.putDouble("timestamp", System.currentTimeMillis());
    
    sendEvent("calendarEventReminder", params);
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
private var listenerCount = 0

private fun sendEvent(eventName: String, params: WritableMap) {
    if (listenerCount > 0) {
        reactApplicationContext
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
            .emit(eventName, params)
    }
}

override fun addListener(eventName: String) {
    listenerCount += 1
}

override fun removeListeners(count: Double) {
    listenerCount -= count.toInt()
    if (listenerCount < 0) {
        listenerCount = 0
    }
}

fun startCalendarEventReminders() {
    // Simulate sending reminders
    val params = Arguments.createMap().apply {
        putString("eventId", "event_123")
        putString("title", "Team Meeting starting in 15 minutes")
        putDouble("timestamp", System.currentTimeMillis().toDouble())
    }
    
    sendEvent("calendarEventReminder", params)
}
```

</TabItem>
</Tabs>

### JavaScript Usage

```javascript
import NativeCalendarModule from './NativeCalendarModule';
import { NativeEventEmitter } from 'react-native';

const calendarEmitter = new NativeEventEmitter(NativeCalendarModule);

// Subscribe to events
const subscription = calendarEmitter.addListener(
  'calendarEventReminder',
  (reminder) => {
    console.log('Reminder:', reminder.title);
    console.log('Event ID:', reminder.eventId);
  }
);

// Start reminders
NativeCalendarModule.startCalendarEventReminders();

// Clean up
subscription.remove();
```

:::info Key Changes
- **Listener tracking**: Must implement `addListener` and `removeListeners` methods
- **Listener count**: Track active listeners to optimize event emission
- **Required methods**: These methods are defined in the TypeScript spec
:::

## Complete Example

Here's the complete migrated module:

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/modules/CalendarModule.java"
package com.yourapp.modules;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableArray;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.Promise;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.facebook.react.turbomodule.core.interfaces.CallbackWrapper;

import com.yourapp.NativeCalendarModuleSpec;

import java.util.Map;
import java.util.HashMap;

@ReactModule(name = CalendarModule.NAME)
public class CalendarModule extends NativeCalendarModuleSpec {
    public static final String NAME = "CalendarModule";
    private int listenerCount = 0;
    
    public CalendarModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    
    @Override
    @NonNull
    public String getName() {
        return NAME;
    }
    
    @Override
    public String getDefaultCalendarName() {
        return "Personal Calendar";
    }
    
    @Override
    public void getCalendarEvents(String startDate, String endDate, CallbackWrapper callback) {
        try {
            WritableArray events = Arguments.createArray();
            
            WritableMap event1 = Arguments.createMap();
            event1.putString("id", "1");
            event1.putString("title", "Team Meeting");
            event1.putString("startDate", startDate);
            event1.putString("endDate", endDate);
            events.pushMap(event1);
            
            WritableMap event2 = Arguments.createMap();
            event2.putString("id", "2");
            event2.putString("title", "Project Review");
            event2.putString("startDate", startDate);
            event2.putString("endDate", endDate);
            events.pushMap(event2);
            
            callback.invoke(null, events);
        } catch (Exception e) {
            callback.invoke(e.getMessage(), null);
        }
    }
    
    @Override
    public void createCalendarEvent(
        String title,
        String startDate,
        String endDate,
        @Nullable String location,
        Promise promise
    ) {
        try {
            String eventId = "event_" + System.currentTimeMillis();
            promise.resolve(eventId);
        } catch (Exception e) {
            promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e);
        }
    }
    
    @Override
    @Nullable
    public WritableMap getCalendarById(String calendarId) {
        if ("primary".equals(calendarId)) {
            WritableMap calendar = Arguments.createMap();
            calendar.putString("id", "primary");
            calendar.putString("title", "Personal Calendar");
            calendar.putBoolean("isPrimary", true);
            return calendar;
        }
        return null;
    }
    
    @Override
    protected Map<String, Object> getTypedExportedConstants() {
        final Map<String, Object> constants = new HashMap<>();
        constants.put("DEFAULT_EVENT_STATUS", "CONFIRMED");
        
        final Map<String, Object> permissions = new HashMap<>();
        permissions.put("READ", "android.permission.READ_CALENDAR");
        permissions.put("WRITE", "android.permission.WRITE_CALENDAR");
        constants.put("CALENDAR_PERMISSIONS", permissions);
        
        return constants;
    }
    
    @Override
    public void addListener(String eventName) {
        listenerCount += 1;
    }
    
    @Override
    public void removeListeners(double count) {
        listenerCount -= (int) count;
        if (listenerCount < 0) {
            listenerCount = 0;
        }
    }
    
    private void sendEvent(String eventName, WritableMap params) {
        if (listenerCount > 0) {
            getReactApplicationContext()
                .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit(eventName, params);
        }
    }
    
    public void startCalendarEventReminders() {
        WritableMap params = Arguments.createMap();
        params.putString("eventId", "event_123");
        params.putString("title", "Team Meeting starting in 15 minutes");
        params.putDouble("timestamp", System.currentTimeMillis());
        
        sendEvent("calendarEventReminder", params);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/modules/CalendarModule.kt"
package com.yourapp.modules

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.bridge.WritableMap
import com.facebook.react.bridge.WritableArray
import com.facebook.react.bridge.Arguments
import com.facebook.react.bridge.Promise
import com.facebook.react.module.annotations.ReactModule
import com.facebook.react.modules.core.DeviceEventManagerModule
import com.facebook.react.turbomodule.core.interfaces.CallbackWrapper

import com.yourapp.NativeCalendarModuleSpec

@ReactModule(name = CalendarModule.NAME)
class CalendarModule(reactContext: ReactApplicationContext) : 
    NativeCalendarModuleSpec(reactContext) {
    
    companion object {
        const val NAME = "CalendarModule"
    }
    
    private var listenerCount = 0
    
    override fun getName(): String = NAME
    
    override fun getDefaultCalendarName(): String {
        return "Personal Calendar"
    }
    
    override fun getCalendarEvents(startDate: String, endDate: String, callback: CallbackWrapper) {
        try {
            val events = Arguments.createArray()
            
            val event1 = Arguments.createMap().apply {
                putString("id", "1")
                putString("title", "Team Meeting")
                putString("startDate", startDate)
                putString("endDate", endDate)
            }
            events.pushMap(event1)
            
            val event2 = Arguments.createMap().apply {
                putString("id", "2")
                putString("title", "Project Review")
                putString("startDate", startDate)
                putString("endDate", endDate)
            }
            events.pushMap(event2)
            
            callback.invoke(null, events)
        } catch (e: Exception) {
            callback.invoke(e.message, null)
        }
    }
    
    override fun createCalendarEvent(
        title: String,
        startDate: String,
        endDate: String,
        location: String?,
        promise: Promise
    ) {
        try {
            val eventId = "event_${System.currentTimeMillis()}"
            promise.resolve(eventId)
        } catch (e: Exception) {
            promise.reject("CREATE_EVENT_ERROR", "Failed to create calendar event", e)
        }
    }
    
    override fun getCalendarById(calendarId: String): WritableMap? {
        return if (calendarId == "primary") {
            Arguments.createMap().apply {
                putString("id", "primary")
                putString("title", "Personal Calendar")
                putBoolean("isPrimary", true)
            }
        } else {
            null
        }
    }
    
    override fun getTypedExportedConstants(): Map<String, Any> {
        return mapOf(
            "DEFAULT_EVENT_STATUS" to "CONFIRMED",
            "CALENDAR_PERMISSIONS" to mapOf(
                "READ" to "android.permission.READ_CALENDAR",
                "WRITE" to "android.permission.WRITE_CALENDAR"
            )
        )
    }
    
    override fun addListener(eventName: String) {
        listenerCount += 1
    }
    
    override fun removeListeners(count: Double) {
        listenerCount -= count.toInt()
        if (listenerCount < 0) {
            listenerCount = 0
        }
    }
    
    private fun sendEvent(eventName: String, params: WritableMap) {
        if (listenerCount > 0) {
            reactApplicationContext
                .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
                .emit(eventName, params)
        }
    }
    
    fun startCalendarEventReminders() {
        val params = Arguments.createMap().apply {
            putString("eventId", "event_123")
            putString("title", "Team Meeting starting in 15 minutes")
            putDouble("timestamp", System.currentTimeMillis().toDouble())
        }
        
        sendEvent("calendarEventReminder", params)
    }
}
```

</TabItem>
</Tabs>

## Summary

Migrating to the new architecture involves:

1. **Creating a TypeScript specification** that defines your module's interface
2. **Extending the generated spec class** instead of `ReactContextBaseJavaModule`
3. **Removing @ReactMethod annotations** as methods are defined in the spec
4. **Implementing listener methods** for event emitters
5. **Ensuring type safety** throughout your implementation

### Benefits of Migration

- **Better Performance**: Direct communication between JavaScript and native code
- **Type Safety**: Compile-time checking prevents runtime errors
- **Code Generation**: Less boilerplate code to maintain
- **Future-Proof**: Aligned with React Native's architectural direction

### Next Steps

- Test your migrated module with both architectures enabled
- Consider migrating your Native Components next
- Update your app's build configuration for the new architecture
- Review the [Turbo Native Modules documentation](/docs/turbo-native-modules-android) for advanced features

Remember to test thoroughly on both old and new architecture to ensure compatibility during the migration period!