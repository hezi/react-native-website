---
id: native-components-migration-android
title: Native Components Migration - Android
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Native Components Migration - Android

This guide will walk you through migrating your existing Android native UI components from the legacy architecture to the new architecture (Fabric). We'll use a `CustomSlider` component to demonstrate the migration process.

## Introduction

The new architecture introduces Fabric, which offers improved performance through synchronous layout, better type safety with code generation, and more efficient memory usage. This guide shows you how to migrate your existing native components step by step.

## Prerequisites

Before starting the migration, ensure you have:

1. A React Native app with the new architecture enabled
2. Basic knowledge of Android view development
3. An existing native component that you want to migrate
4. TypeScript or Flow set up in your project

## CustomSlider Component Overview

Our example `CustomSlider` component demonstrates migration of:

- View properties (props)
- Event handling
- Commands (imperative methods)
- Style and layout props
- Accessibility features

The component provides:
- Value tracking with min/max bounds
- Custom styling for thumb and track colors
- Events for value changes and sliding states
- Programmatic control through commands

## Project Structure Overview

Here's how the structure changes between legacy and new architecture:

### Legacy Architecture
```
android/app/src/main/java/com/yourapp/
├── MainApplication.java
└── customslider/
    ├── CustomSliderView.java
    ├── CustomSliderManager.java
    └── CustomSliderPackage.java
```

### New Architecture
```
android/app/src/main/java/com/yourapp/
├── MainApplication.java
└── customslider/
    ├── CustomSliderView.java
    ├── CustomSliderManager.java
    └── CustomSliderPackage.java

js/
└── CustomSliderNativeComponent.ts  # TypeScript specification
```

## Step 1: Creating the Fabric Component Specification

The first step is to create a TypeScript specification that defines the interface of your native component.

<Tabs groupId="js-language">
<TabItem value="ts" label="TypeScript">

```typescript title="js/CustomSliderNativeComponent.ts"
import type {ViewProps} from 'react-native';
import type {HostComponent} from 'react-native';
import type {
  DirectEventHandler,
  Float,
  Int32,
} from 'react-native/Libraries/Types/CodegenTypes';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import codegenNativeCommands from 'react-native/Libraries/Utilities/codegenNativeCommands';

export type OnValueChangeEvent = Readonly<{
  value: Float;
}>;

export type OnSlidingEvent = Readonly<{
  value: Float;
}>;

export interface NativeProps extends ViewProps {
  // Basic props
  value?: Float;
  minimumValue?: Float;
  maximumValue?: Float;
  step?: Float;
  
  // Style props
  thumbTintColor?: string;
  minimumTrackTintColor?: string;
  maximumTrackTintColor?: string;
  
  // Events
  onValueChange?: DirectEventHandler<OnValueChangeEvent>;
  onSlidingStart?: DirectEventHandler<OnSlidingEvent>;
  onSlidingComplete?: DirectEventHandler<OnSlidingEvent>;
}

export interface NativeCommands {
  setValue: (viewRef: React.ElementRef<HostComponent<NativeProps>>, value: Float) => void;
  setValueAnimated: (viewRef: React.ElementRef<HostComponent<NativeProps>>, value: Float, duration: Int32) => void;
}

export const Commands = codegenNativeCommands<NativeCommands>({
  supportedCommands: ['setValue', 'setValueAnimated'],
});

export default codegenNativeComponent<NativeProps>('CustomSlider');
```

</TabItem>
<TabItem value="flow" label="Flow">

```javascript title="js/CustomSliderNativeComponent.js"
// @flow

import type {ViewProps} from 'react-native';
import type {HostComponent} from 'react-native';
import type {
  DirectEventHandler,
  Float,
  Int32,
} from 'react-native/Libraries/Types/CodegenTypes';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';
import codegenNativeCommands from 'react-native/Libraries/Utilities/codegenNativeCommands';

export type OnValueChangeEvent = $ReadOnly<{|
  value: Float,
|}>;

export type OnSlidingEvent = $ReadOnly<{|
  value: Float,
|}>;

export type NativeProps = $ReadOnly<{|
  ...ViewProps,
  
  // Basic props
  value?: Float,
  minimumValue?: Float,
  maximumValue?: Float,
  step?: Float,
  
  // Style props
  thumbTintColor?: string,
  minimumTrackTintColor?: string,
  maximumTrackTintColor?: string,
  
  // Events
  onValueChange?: DirectEventHandler<OnValueChangeEvent>,
  onSlidingStart?: DirectEventHandler<OnSlidingEvent>,
  onSlidingComplete?: DirectEventHandler<OnSlidingEvent>,
|}>;

export type NativeCommands = {|
  +setValue: (viewRef: React.ElementRef<HostComponent<NativeProps>>, value: Float) => void,
  +setValueAnimated: (viewRef: React.ElementRef<HostComponent<NativeProps>>, value: Float, duration: Int32) => void,
|};

export const Commands = codegenNativeCommands<NativeCommands>({
  supportedCommands: ['setValue', 'setValueAnimated'],
});

export default codegenNativeComponent<NativeProps>('CustomSlider');
```

</TabItem>
</Tabs>

## Step 2: Basic View Manager Setup

Let's look at how the basic ViewManager setup changes between architectures.

### Legacy Architecture

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
package com.yourapp.customslider;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.annotations.ReactProp;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.annotations.ReactModule;

@ReactModule(name = CustomSliderManager.REACT_CLASS)
public class CustomSliderManager extends SimpleViewManager<CustomSliderView> {
    public static final String REACT_CLASS = "CustomSlider";
    
    private final ReactApplicationContext mCallerContext;
    
    public CustomSliderManager(ReactApplicationContext reactContext) {
        mCallerContext = reactContext;
    }
    
    @NonNull
    @Override
    public String getName() {
        return REACT_CLASS;
    }
    
    @NonNull
    @Override
    protected CustomSliderView createViewInstance(@NonNull ThemedReactContext reactContext) {
        return new CustomSliderView(reactContext);
    }
    
    // Props and methods implementation continues...
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
package com.yourapp.customslider

import com.facebook.react.uimanager.SimpleViewManager
import com.facebook.react.uimanager.ThemedReactContext
import com.facebook.react.uimanager.annotations.ReactProp
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.module.annotations.ReactModule

@ReactModule(name = CustomSliderManager.REACT_CLASS)
class CustomSliderManager(
    private val callerContext: ReactApplicationContext
) : SimpleViewManager<CustomSliderView>() {
    
    companion object {
        const val REACT_CLASS = "CustomSlider"
    }
    
    override fun getName(): String = REACT_CLASS
    
    override fun createViewInstance(reactContext: ThemedReactContext): CustomSliderView {
        return CustomSliderView(reactContext)
    }
    
    // Props and methods implementation continues...
}
```

</TabItem>
</Tabs>

### New Architecture

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
package com.yourapp.customslider;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.ViewManagerDelegate;
import com.facebook.react.viewmanagers.CustomSliderManagerDelegate;
import com.facebook.react.viewmanagers.CustomSliderManagerInterface;

@ReactModule(name = CustomSliderManager.REACT_CLASS)
public class CustomSliderManager extends SimpleViewManager<CustomSliderView>
        implements CustomSliderManagerInterface<CustomSliderView> {
    
    public static final String REACT_CLASS = "CustomSlider";
    
    private final ViewManagerDelegate<CustomSliderView> mDelegate;
    
    public CustomSliderManager(ReactApplicationContext reactContext) {
        mDelegate = new CustomSliderManagerDelegate(this);
    }
    
    @Nullable
    @Override
    protected ViewManagerDelegate<CustomSliderView> getDelegate() {
        return mDelegate;
    }
    
    @NonNull
    @Override
    public String getName() {
        return REACT_CLASS;
    }
    
    @NonNull
    @Override
    protected CustomSliderView createViewInstance(@NonNull ThemedReactContext reactContext) {
        return new CustomSliderView(reactContext);
    }
    
    // Interface methods implementation continues...
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
package com.yourapp.customslider

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.module.annotations.ReactModule
import com.facebook.react.uimanager.SimpleViewManager
import com.facebook.react.uimanager.ThemedReactContext
import com.facebook.react.uimanager.ViewManagerDelegate
import com.facebook.react.viewmanagers.CustomSliderManagerDelegate
import com.facebook.react.viewmanagers.CustomSliderManagerInterface

@ReactModule(name = CustomSliderManager.REACT_CLASS)
class CustomSliderManager(
    reactContext: ReactApplicationContext
) : SimpleViewManager<CustomSliderView>(), 
    CustomSliderManagerInterface<CustomSliderView> {
    
    companion object {
        const val REACT_CLASS = "CustomSlider"
    }
    
    private val delegate = CustomSliderManagerDelegate(this)
    
    override fun getDelegate(): ViewManagerDelegate<CustomSliderView> = delegate
    
    override fun getName(): String = REACT_CLASS
    
    override fun createViewInstance(reactContext: ThemedReactContext): CustomSliderView {
        return CustomSliderView(reactContext)
    }
    
    // Interface methods implementation continues...
}
```

</TabItem>
</Tabs>

:::info Key Changes
- **Interface Implementation**: ViewManager now implements a generated interface
- **Delegate Pattern**: Uses ViewManagerDelegate for prop setting
- **Type Safety**: Generated code ensures type safety for props
:::

## Step 3: Migrating Props

Props migration involves changing from `@ReactProp` annotations to interface methods.

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
@ReactProp(name = "value", defaultFloat = 0f)
public void setValue(CustomSliderView view, float value) {
    view.setValue(value);
}

@ReactProp(name = "minimumValue", defaultFloat = 0f)
public void setMinimumValue(CustomSliderView view, float minimumValue) {
    view.setMinimumValue(minimumValue);
}

@ReactProp(name = "maximumValue", defaultFloat = 1f)
public void setMaximumValue(CustomSliderView view, float maximumValue) {
    view.setMaximumValue(maximumValue);
}

@ReactProp(name = "step", defaultFloat = 0f)
public void setStep(CustomSliderView view, float step) {
    view.setStep(step);
}

@ReactProp(name = "thumbTintColor", customType = "Color")
public void setThumbTintColor(CustomSliderView view, @Nullable Integer color) {
    view.setThumbTintColor(color);
}

@ReactProp(name = "minimumTrackTintColor", customType = "Color")
public void setMinimumTrackTintColor(CustomSliderView view, @Nullable Integer color) {
    view.setMinimumTrackTintColor(color);
}

@ReactProp(name = "maximumTrackTintColor", customType = "Color")
public void setMaximumTrackTintColor(CustomSliderView view, @Nullable Integer color) {
    view.setMaximumTrackTintColor(color);
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
@ReactProp(name = "value", defaultFloat = 0f)
fun setValue(view: CustomSliderView, value: Float) {
    view.setValue(value)
}

@ReactProp(name = "minimumValue", defaultFloat = 0f)
fun setMinimumValue(view: CustomSliderView, minimumValue: Float) {
    view.setMinimumValue(minimumValue)
}

@ReactProp(name = "maximumValue", defaultFloat = 1f)
fun setMaximumValue(view: CustomSliderView, maximumValue: Float) {
    view.setMaximumValue(maximumValue)
}

@ReactProp(name = "step", defaultFloat = 0f)
fun setStep(view: CustomSliderView, step: Float) {
    view.setStep(step)
}

@ReactProp(name = "thumbTintColor", customType = "Color")
fun setThumbTintColor(view: CustomSliderView, color: Int?) {
    view.setThumbTintColor(color)
}

@ReactProp(name = "minimumTrackTintColor", customType = "Color")
fun setMinimumTrackTintColor(view: CustomSliderView, color: Int?) {
    view.setMinimumTrackTintColor(color)
}

@ReactProp(name = "maximumTrackTintColor", customType = "Color")
fun setMaximumTrackTintColor(view: CustomSliderView, color: Int?) {
    view.setMaximumTrackTintColor(color)
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
@Override
public void setValue(CustomSliderView view, float value) {
    view.setValue(value);
}

@Override
public void setMinimumValue(CustomSliderView view, float minimumValue) {
    view.setMinimumValue(minimumValue);
}

@Override
public void setMaximumValue(CustomSliderView view, float maximumValue) {
    view.setMaximumValue(maximumValue);
}

@Override
public void setStep(CustomSliderView view, float step) {
    view.setStep(step);
}

@Override
public void setThumbTintColor(CustomSliderView view, @Nullable String color) {
    view.setThumbTintColor(ColorPropConverter.getColor(color, view.getContext()));
}

@Override
public void setMinimumTrackTintColor(CustomSliderView view, @Nullable String color) {
    view.setMinimumTrackTintColor(ColorPropConverter.getColor(color, view.getContext()));
}

@Override
public void setMaximumTrackTintColor(CustomSliderView view, @Nullable String color) {
    view.setMaximumTrackTintColor(ColorPropConverter.getColor(color, view.getContext()));
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
override fun setValue(view: CustomSliderView, value: Float) {
    view.setValue(value)
}

override fun setMinimumValue(view: CustomSliderView, minimumValue: Float) {
    view.setMinimumValue(minimumValue)
}

override fun setMaximumValue(view: CustomSliderView, maximumValue: Float) {
    view.setMaximumValue(maximumValue)
}

override fun setStep(view: CustomSliderView, step: Float) {
    view.setStep(step)
}

override fun setThumbTintColor(view: CustomSliderView, color: String?) {
    view.setThumbTintColor(ColorPropConverter.getColor(color, view.context))
}

override fun setMinimumTrackTintColor(view: CustomSliderView, color: String?) {
    view.setMinimumTrackTintColor(ColorPropConverter.getColor(color, view.context))
}

override fun setMaximumTrackTintColor(view: CustomSliderView, color: String?) {
    view.setMaximumTrackTintColor(ColorPropConverter.getColor(color, view.context))
}
```

</TabItem>
</Tabs>

:::info Key Changes
- **No @ReactProp annotations**: Props are defined in the TypeScript spec
- **Interface methods**: Implement methods from generated interface
- **Color handling**: Colors come as strings and need conversion
:::

## Step 4: Migrating Events

Event handling changes significantly in the new architecture with better type safety.

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
@Nullable
@Override
public Map<String, Object> getExportedCustomBubblingEventTypeConstants() {
    return MapBuilder.<String, Object>builder()
        .put("onValueChange",
            MapBuilder.of(
                "phasedRegistrationNames",
                MapBuilder.of("bubbled", "onValueChange")))
        .put("onSlidingStart",
            MapBuilder.of(
                "phasedRegistrationNames",
                MapBuilder.of("bubbled", "onSlidingStart")))
        .put("onSlidingComplete",
            MapBuilder.of(
                "phasedRegistrationNames",
                MapBuilder.of("bubbled", "onSlidingComplete")))
        .build();
}

// In CustomSliderView.java
private void emitValueChangeEvent(float value) {
    WritableMap event = Arguments.createMap();
    event.putDouble("value", value);
    
    ReactContext reactContext = (ReactContext) getContext();
    reactContext.getJSModule(RCTEventEmitter.class)
        .receiveEvent(getId(), "onValueChange", event);
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
override fun getExportedCustomBubblingEventTypeConstants(): Map<String, Any>? {
    return mapOf(
        "onValueChange" to mapOf(
            "phasedRegistrationNames" to mapOf("bubbled" to "onValueChange")
        ),
        "onSlidingStart" to mapOf(
            "phasedRegistrationNames" to mapOf("bubbled" to "onSlidingStart")
        ),
        "onSlidingComplete" to mapOf(
            "phasedRegistrationNames" to mapOf("bubbled" to "onSlidingComplete")
        )
    )
}

// In CustomSliderView.kt
private fun emitValueChangeEvent(value: Float) {
    val event = Arguments.createMap().apply {
        putDouble("value", value.toDouble())
    }
    
    val reactContext = context as ReactContext
    reactContext.getJSModule(RCTEventEmitter::class.java)
        .receiveEvent(id, "onValueChange", event)
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderView.java"
package com.yourapp.customslider;

import android.content.Context;
import android.widget.SeekBar;

import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.uimanager.UIManagerHelper;
import com.facebook.react.uimanager.events.Event;
import com.facebook.react.uimanager.events.EventDispatcher;
import com.facebook.react.uimanager.events.RCTEventEmitter;

public class CustomSliderView extends SeekBar {
    
    public CustomSliderView(Context context) {
        super(context);
        setupListeners();
    }
    
    private void setupListeners() {
        setOnSeekBarChangeListener(new OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                if (fromUser) {
                    float value = progressToValue(progress);
                    emitOnValueChangeEvent(value);
                }
            }
            
            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {
                float value = progressToValue(seekBar.getProgress());
                emitOnSlidingStartEvent(value);
            }
            
            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                float value = progressToValue(seekBar.getProgress());
                emitOnSlidingCompleteEvent(value);
            }
        });
    }
    
    private void emitOnValueChangeEvent(float value) {
        ReactContext reactContext = (ReactContext) getContext();
        int surfaceId = UIManagerHelper.getSurfaceId(this);
        EventDispatcher eventDispatcher = UIManagerHelper.getEventDispatcherForReactTag(
            reactContext, getId());
        
        if (eventDispatcher != null) {
            WritableMap eventData = Arguments.createMap();
            eventData.putDouble("value", value);
            
            Event event = new Event(surfaceId, getId()) {
                @Override
                public String getEventName() {
                    return "onValueChange";
                }
                
                @Override
                protected WritableMap getEventData() {
                    return eventData;
                }
            };
            
            eventDispatcher.dispatchEvent(event);
        }
    }
    
    // Similar methods for onSlidingStart and onSlidingComplete...
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderView.kt"
package com.yourapp.customslider

import android.content.Context
import android.widget.SeekBar
import com.facebook.react.bridge.Arguments
import com.facebook.react.bridge.ReactContext
import com.facebook.react.bridge.WritableMap
import com.facebook.react.uimanager.UIManagerHelper
import com.facebook.react.uimanager.events.Event
import com.facebook.react.uimanager.events.EventDispatcher

class CustomSliderView(context: Context) : SeekBar(context) {
    
    init {
        setupListeners()
    }
    
    private fun setupListeners() {
        setOnSeekBarChangeListener(object : OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                if (fromUser) {
                    val value = progressToValue(progress)
                    emitOnValueChangeEvent(value)
                }
            }
            
            override fun onStartTrackingTouch(seekBar: SeekBar?) {
                val value = progressToValue(seekBar?.progress ?: 0)
                emitOnSlidingStartEvent(value)
            }
            
            override fun onStopTrackingTouch(seekBar: SeekBar?) {
                val value = progressToValue(seekBar?.progress ?: 0)
                emitOnSlidingCompleteEvent(value)
            }
        })
    }
    
    private fun emitOnValueChangeEvent(value: Float) {
        val reactContext = context as ReactContext
        val surfaceId = UIManagerHelper.getSurfaceId(this)
        val eventDispatcher = UIManagerHelper.getEventDispatcherForReactTag(
            reactContext, id)
        
        eventDispatcher?.let {
            val eventData = Arguments.createMap().apply {
                putDouble("value", value.toDouble())
            }
            
            val event = object : Event<Event<*>>(surfaceId, id) {
                override fun getEventName(): String = "onValueChange"
                
                override fun getEventData(): WritableMap? = eventData
            }
            
            it.dispatchEvent(event)
        }
    }
    
    // Similar methods for onSlidingStart and onSlidingComplete...
}
```

</TabItem>
</Tabs>

:::info Key Changes
- **Event registration**: No need to register events manually
- **Type-safe events**: Event structure defined in TypeScript spec
- **New event API**: Uses EventDispatcher with surface ID
:::

## Step 5: Migrating Commands

Commands allow JavaScript to imperatively control the native view.

### Legacy Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
@Override
public Map<String, Integer> getCommandsMap() {
    return MapBuilder.of(
        "setValue", COMMAND_SET_VALUE,
        "setValueAnimated", COMMAND_SET_VALUE_ANIMATED
    );
}

@Override
public void receiveCommand(
    @NonNull CustomSliderView view,
    int commandId,
    @Nullable ReadableArray args
) {
    switch (commandId) {
        case COMMAND_SET_VALUE:
            if (args != null) {
                float value = (float) args.getDouble(0);
                view.setValue(value);
            }
            break;
        case COMMAND_SET_VALUE_ANIMATED:
            if (args != null) {
                float value = (float) args.getDouble(0);
                int duration = args.getInt(1);
                view.setValueAnimated(value, duration);
            }
            break;
    }
}

private static final int COMMAND_SET_VALUE = 1;
private static final int COMMAND_SET_VALUE_ANIMATED = 2;
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
override fun getCommandsMap(): Map<String, Int> {
    return mapOf(
        "setValue" to COMMAND_SET_VALUE,
        "setValueAnimated" to COMMAND_SET_VALUE_ANIMATED
    )
}

override fun receiveCommand(
    view: CustomSliderView,
    commandId: Int,
    args: ReadableArray?
) {
    when (commandId) {
        COMMAND_SET_VALUE -> {
            args?.let {
                val value = it.getDouble(0).toFloat()
                view.setValue(value)
            }
        }
        COMMAND_SET_VALUE_ANIMATED -> {
            args?.let {
                val value = it.getDouble(0).toFloat()
                val duration = it.getInt(1)
                view.setValueAnimated(value, duration)
            }
        }
    }
}

companion object {
    private const val COMMAND_SET_VALUE = 1
    private const val COMMAND_SET_VALUE_ANIMATED = 2
}
```

</TabItem>
</Tabs>

### New Architecture Implementation

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
@Override
public void setValue(CustomSliderView view, float value) {
    view.setValue(value);
}

@Override
public void setValueAnimated(CustomSliderView view, float value, int duration) {
    view.setValueAnimated(value, duration);
}

// No more getCommandsMap or receiveCommand methods needed!
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
override fun setValue(view: CustomSliderView, value: Float) {
    view.setValue(value)
}

override fun setValueAnimated(view: CustomSliderView, value: Float, duration: Int) {
    view.setValueAnimated(value, duration)
}

// No more getCommandsMap or receiveCommand methods needed!
```

</TabItem>
</Tabs>

:::info Key Changes
- **Direct methods**: Commands are now regular interface methods
- **Type safety**: Parameters are strongly typed
- **No command IDs**: No need for command maps or switch statements
:::

## Step 6: Style and Layout Props

The new architecture handles style and layout props automatically through the base implementation.

### JavaScript Wrapper Component

Here's how to create a JavaScript wrapper for your native component:

```javascript title="CustomSlider.js"
import React, { useRef } from 'react';
import { 
  requireNativeComponent,
  UIManager,
  findNodeHandle,
  StyleSheet
} from 'react-native';
import CustomSliderNativeComponent, { Commands } from './CustomSliderNativeComponent';

const CustomSlider = (props) => {
  const nativeRef = useRef(null);
  
  const setValue = (value) => {
    if (nativeRef.current) {
      Commands.setValue(nativeRef.current, value);
    }
  };
  
  const setValueAnimated = (value, duration = 300) => {
    if (nativeRef.current) {
      Commands.setValueAnimated(nativeRef.current, value, duration);
    }
  };
  
  // Expose imperative methods
  React.useImperativeHandle(ref, () => ({
    setValue,
    setValueAnimated,
  }));
  
  return (
    <CustomSliderNativeComponent
      {...props}
      ref={nativeRef}
      style={[styles.default, props.style]}
    />
  );
};

const styles = StyleSheet.create({
  default: {
    height: 40,
  },
});

export default React.forwardRef(CustomSlider);
```

### Complete ViewManager Example

Here's the complete migrated ViewManager:

<Tabs groupId="android-language">
<TabItem value="java" label="Java">

```java title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.java"
package com.yourapp.customslider;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.module.annotations.ReactModule;
import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.ViewManagerDelegate;
import com.facebook.react.uimanager.annotations.ReactProp;
import com.facebook.react.common.MapBuilder;
import com.facebook.react.viewmanagers.CustomSliderManagerDelegate;
import com.facebook.react.viewmanagers.CustomSliderManagerInterface;

import java.util.Map;

@ReactModule(name = CustomSliderManager.REACT_CLASS)
public class CustomSliderManager extends SimpleViewManager<CustomSliderView>
        implements CustomSliderManagerInterface<CustomSliderView> {
    
    public static final String REACT_CLASS = "CustomSlider";
    
    private final ViewManagerDelegate<CustomSliderView> mDelegate;
    
    public CustomSliderManager(ReactApplicationContext reactContext) {
        mDelegate = new CustomSliderManagerDelegate(this);
    }
    
    @Nullable
    @Override
    protected ViewManagerDelegate<CustomSliderView> getDelegate() {
        return mDelegate;
    }
    
    @NonNull
    @Override
    public String getName() {
        return REACT_CLASS;
    }
    
    @NonNull
    @Override
    protected CustomSliderView createViewInstance(@NonNull ThemedReactContext reactContext) {
        return new CustomSliderView(reactContext);
    }
    
    @Override
    public void setValue(CustomSliderView view, float value) {
        view.setValue(value);
    }
    
    @Override
    public void setMinimumValue(CustomSliderView view, float minimumValue) {
        view.setMinimumValue(minimumValue);
    }
    
    @Override
    public void setMaximumValue(CustomSliderView view, float maximumValue) {
        view.setMaximumValue(maximumValue);
    }
    
    @Override
    public void setStep(CustomSliderView view, float step) {
        view.setStep(step);
    }
    
    @Override
    public void setThumbTintColor(CustomSliderView view, @Nullable String color) {
        view.setThumbTintColor(ColorPropConverter.getColor(color, view.getContext()));
    }
    
    @Override
    public void setMinimumTrackTintColor(CustomSliderView view, @Nullable String color) {
        view.setMinimumTrackTintColor(ColorPropConverter.getColor(color, view.getContext()));
    }
    
    @Override
    public void setMaximumTrackTintColor(CustomSliderView view, @Nullable String color) {
        view.setMaximumTrackTintColor(ColorPropConverter.getColor(color, view.getContext()));
    }
    
    @Override
    public void setValue(CustomSliderView view, float value) {
        view.setValue(value);
    }
    
    @Override
    public void setValueAnimated(CustomSliderView view, float value, int duration) {
        view.setValueAnimated(value, duration);
    }
}
```

</TabItem>
<TabItem value="kotlin" label="Kotlin">

```kotlin title="android/app/src/main/java/com/yourapp/customslider/CustomSliderManager.kt"
package com.yourapp.customslider

import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.module.annotations.ReactModule
import com.facebook.react.uimanager.SimpleViewManager
import com.facebook.react.uimanager.ThemedReactContext
import com.facebook.react.uimanager.ViewManagerDelegate
import com.facebook.react.viewmanagers.CustomSliderManagerDelegate
import com.facebook.react.viewmanagers.CustomSliderManagerInterface

@ReactModule(name = CustomSliderManager.REACT_CLASS)
class CustomSliderManager(
    reactContext: ReactApplicationContext
) : SimpleViewManager<CustomSliderView>(), 
    CustomSliderManagerInterface<CustomSliderView> {
    
    companion object {
        const val REACT_CLASS = "CustomSlider"
    }
    
    private val delegate = CustomSliderManagerDelegate(this)
    
    override fun getDelegate(): ViewManagerDelegate<CustomSliderView> = delegate
    
    override fun getName(): String = REACT_CLASS
    
    override fun createViewInstance(reactContext: ThemedReactContext): CustomSliderView {
        return CustomSliderView(reactContext)
    }
    
    override fun setValue(view: CustomSliderView, value: Float) {
        view.setValue(value)
    }
    
    override fun setMinimumValue(view: CustomSliderView, minimumValue: Float) {
        view.setMinimumValue(minimumValue)
    }
    
    override fun setMaximumValue(view: CustomSliderView, maximumValue: Float) {
        view.setMaximumValue(maximumValue)
    }
    
    override fun setStep(view: CustomSliderView, step: Float) {
        view.setStep(step)
    }
    
    override fun setThumbTintColor(view: CustomSliderView, color: String?) {
        view.setThumbTintColor(ColorPropConverter.getColor(color, view.context))
    }
    
    override fun setMinimumTrackTintColor(view: CustomSliderView, color: String?) {
        view.setMinimumTrackTintColor(ColorPropConverter.getColor(color, view.context))
    }
    
    override fun setMaximumTrackTintColor(view: CustomSliderView, color: String?) {
        view.setMaximumTrackTintColor(ColorPropConverter.getColor(color, view.context))
    }
    
    override fun setValue(view: CustomSliderView, value: Float) {
        view.setValue(value)
    }
    
    override fun setValueAnimated(view: CustomSliderView, value: Float, duration: Int) {
        view.setValueAnimated(value, duration)
    }
}
```

</TabItem>
</Tabs>

## Summary

Migrating to the new architecture involves:

1. **Creating a TypeScript specification** that defines your component's interface
2. **Implementing generated interfaces** instead of using annotations
3. **Using ViewManagerDelegate** for prop management
4. **Updating event handling** to use the new event system
5. **Converting commands** to direct interface methods
6. **Leveraging type safety** throughout your implementation

### Benefits of Migration

- **Better Performance**: Synchronous layout and rendering
- **Type Safety**: Compile-time checking prevents runtime errors
- **Less Boilerplate**: Code generation reduces manual work
- **Better Memory Management**: More efficient view recycling
- **Future-Proof**: Aligned with React Native's architectural direction

### Next Steps

- Run Codegen to generate the native interfaces
- Test your migrated component with both architectures
- Consider migrating your iOS components next
- Update your app's build configuration for the new architecture
- Review the [Fabric Native Components documentation](/docs/fabric-native-components-android) for advanced features

Remember to test thoroughly on both old and new architecture to ensure compatibility during the migration period!