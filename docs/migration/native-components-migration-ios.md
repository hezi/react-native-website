---
id: native-components-migration-ios
title: Native Components Migration - iOS
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Native Components Migration - iOS

This guide will walk you through migrating your existing iOS native UI components from the legacy architecture to the new architecture (Fabric). We'll use a `CustomSlider` component to demonstrate the migration process.

## Introduction

The new architecture on iOS brings the same benefits as Android: improved performance through synchronous layout, better type safety with code generation, and more efficient memory usage. This guide shows you how to migrate your existing iOS native components step by step.

## Prerequisites

Before starting the migration, ensure you have:

1. A React Native app with the new architecture enabled
2. Basic knowledge of iOS view development (UIKit)
3. An existing native component that you want to migrate
4. TypeScript or Flow set up in your project

## CustomSlider Component Overview

Our example `CustomSlider` component (same as Android) demonstrates migration of:

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
ios/YourApp/
├── CustomSliderManager.h
├── CustomSliderManager.m
├── CustomSliderView.h
└── CustomSliderView.m
```

### New Architecture
```
ios/YourApp/
├── CustomSliderManager.h
├── CustomSliderManager.mm  # Note the .mm extension
├── CustomSliderView.h
├── CustomSliderView.mm     # Note the .mm extension
└── CustomSliderComponentView.h  # New Fabric component view
└── CustomSliderComponentView.mm

js/
└── CustomSliderNativeComponent.ts  # TypeScript specification (same as Android)
```

## Step 1: Creating the Fabric Component Specification

The TypeScript specification is identical to the Android version. This is one of the benefits of the new architecture - you write the spec once and use it for both platforms.

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

```objc title="ios/YourApp/CustomSliderManager.h"
#import <React/RCTViewManager.h>

@interface CustomSliderManager : RCTViewManager

@end
```

```objc title="ios/YourApp/CustomSliderManager.m"
#import "CustomSliderManager.h"
#import "CustomSliderView.h"

@implementation CustomSliderManager

RCT_EXPORT_MODULE(CustomSlider)

- (UIView *)view
{
  return [[CustomSliderView alloc] init];
}

// Props and methods implementation continues...

@end
```

### New Architecture

```objc title="ios/YourApp/CustomSliderManager.h"
#import <React/RCTViewManager.h>
#import <React/RCTViewComponentView.h>

@interface CustomSliderManager : RCTViewManager

@end
```

```objc title="ios/YourApp/CustomSliderManager.mm"
#import "CustomSliderManager.h"
#import "CustomSliderComponentView.h"

#import <react/renderer/components/CustomSliderNativeComponent/ComponentDescriptors.h>
#import <react/renderer/components/CustomSliderNativeComponent/EventEmitters.h>
#import <react/renderer/components/CustomSliderNativeComponent/Props.h>
#import <react/renderer/components/CustomSliderNativeComponent/RCTComponentViewHelpers.h>

#import "RCTFabricComponentsPlugins.h"

@implementation CustomSliderManager

RCT_EXPORT_MODULE(CustomSlider)

- (UIView *)view
{
  return [[CustomSliderComponentView alloc] init];
}

// Props will be handled by the ComponentView, not the manager

@end
```

:::info Key Changes
- **Implementation file**: Changed from `.m` to `.mm` for Objective-C++ support
- **Imports**: Now imports generated headers from codegen
- **View creation**: Returns a ComponentView instead of a plain view
- **Props handling**: Props are handled in ComponentView, not ViewManager
:::

## Step 3: Creating the ComponentView

The new architecture requires a ComponentView that handles props and state updates.

### New Architecture ComponentView

```objc title="ios/YourApp/CustomSliderComponentView.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface CustomSliderComponentView : RCTViewComponentView

@end

NS_ASSUME_NONNULL_END
```

```objc title="ios/YourApp/CustomSliderComponentView.mm"
#import "CustomSliderComponentView.h"

#import <react/renderer/components/CustomSliderNativeComponent/ComponentDescriptors.h>
#import <react/renderer/components/CustomSliderNativeComponent/EventEmitters.h>
#import <react/renderer/components/CustomSliderNativeComponent/Props.h>
#import <react/renderer/components/CustomSliderNativeComponent/RCTComponentViewHelpers.h>

#import <React/RCTConversions.h>
#import "RCTFabricComponentsPlugins.h"

using namespace facebook::react;

@interface CustomSliderComponentView () <RCTCustomSliderViewProtocol>
@end

@implementation CustomSliderComponentView {
  UISlider *_slider;
  BOOL _isSliding;
}

+ (ComponentDescriptorProvider)componentDescriptorProvider
{
  return concreteComponentDescriptorProvider<CustomSliderComponentDescriptor>();
}

- (instancetype)initWithFrame:(CGRect)frame
{
  if (self = [super initWithFrame:frame]) {
    static const auto defaultProps = std::make_shared<const CustomSliderProps>();
    _props = defaultProps;
    
    _slider = [[UISlider alloc] init];
    _slider.continuous = YES;
    
    [_slider addTarget:self
                action:@selector(onChange:)
      forControlEvents:UIControlEventValueChanged];
    
    [_slider addTarget:self
                action:@selector(onSlidingStart:)
      forControlEvents:UIControlEventTouchDown];
    
    [_slider addTarget:self
                action:@selector(onSlidingComplete:)
      forControlEvents:UIControlEventTouchUpInside | UIControlEventTouchUpOutside];
    
    self.contentView = _slider;
  }
  
  return self;
}

#pragma mark - RCTComponentViewProtocol

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps
{
  const auto &oldViewProps = *std::static_pointer_cast<const CustomSliderProps>(_props);
  const auto &newViewProps = *std::static_pointer_cast<const CustomSliderProps>(props);
  
  if (oldViewProps.value != newViewProps.value) {
    _slider.value = newViewProps.value;
  }
  
  if (oldViewProps.minimumValue != newViewProps.minimumValue) {
    _slider.minimumValue = newViewProps.minimumValue;
  }
  
  if (oldViewProps.maximumValue != newViewProps.maximumValue) {
    _slider.maximumValue = newViewProps.maximumValue;
  }
  
  // Handle color props
  if (oldViewProps.thumbTintColor != newViewProps.thumbTintColor) {
    _slider.thumbTintColor = RCTUIColorFromSharedColor(newViewProps.thumbTintColor);
  }
  
  if (oldViewProps.minimumTrackTintColor != newViewProps.minimumTrackTintColor) {
    _slider.minimumTrackTintColor = RCTUIColorFromSharedColor(newViewProps.minimumTrackTintColor);
  }
  
  if (oldViewProps.maximumTrackTintColor != newViewProps.maximumTrackTintColor) {
    _slider.maximumTrackTintColor = RCTUIColorFromSharedColor(newViewProps.maximumTrackTintColor);
  }
  
  [super updateProps:props oldProps:oldProps];
}

@end
```

:::info Key Changes
- **ComponentView**: New class that extends RCTViewComponentView
- **Props handling**: Props are strongly typed C++ objects
- **Event handling**: Events are dispatched through EventEmitter
- **State management**: Can handle local state efficiently
:::

## Step 4: Migrating Props

Props migration involves moving from RCT_EXPORT_VIEW_PROPERTY macros to ComponentView prop handling.

### Legacy Implementation

```objc title="ios/YourApp/CustomSliderManager.m"
RCT_EXPORT_VIEW_PROPERTY(value, float)
RCT_EXPORT_VIEW_PROPERTY(minimumValue, float)
RCT_EXPORT_VIEW_PROPERTY(maximumValue, float)
RCT_EXPORT_VIEW_PROPERTY(step, float)
RCT_EXPORT_VIEW_PROPERTY(thumbTintColor, UIColor)
RCT_EXPORT_VIEW_PROPERTY(minimumTrackTintColor, UIColor)
RCT_EXPORT_VIEW_PROPERTY(maximumTrackTintColor, UIColor)

// In CustomSliderView.m
@implementation CustomSliderView

- (void)setValue:(float)value
{
  _slider.value = value;
}

- (void)setMinimumValue:(float)minimumValue
{
  _slider.minimumValue = minimumValue;
}

// ... other setters
```

### New Architecture Implementation

Props are now handled in the ComponentView's updateProps method (shown above). The ViewManager no longer exports properties.

:::info Key Changes
- **No RCT_EXPORT_VIEW_PROPERTY macros**: Props are defined in TypeScript spec
- **Type-safe props**: Props are strongly typed C++ objects
- **Efficient updates**: Only changed props are updated
:::

## Step 5: Migrating Events

Event handling changes significantly with the new EventEmitter system.

### Legacy Implementation

```objc title="ios/YourApp/CustomSliderView.m"
@implementation CustomSliderView

- (void)onChange:(UISlider *)sender
{
  if (self.onValueChange) {
    self.onValueChange(@{
      @"value": @(sender.value)
    });
  }
}

- (void)onSlidingStart:(UISlider *)sender
{
  if (self.onSlidingStart) {
    self.onSlidingStart(@{
      @"value": @(sender.value)
    });
  }
}

- (void)onSlidingComplete:(UISlider *)sender
{
  if (self.onSlidingComplete) {
    self.onSlidingComplete(@{
      @"value": @(sender.value)
    });
  }
}

@end
```

### New Architecture Implementation

```objc title="ios/YourApp/CustomSliderComponentView.mm"
#pragma mark - Event Handlers

- (void)onChange:(UISlider *)sender
{
  if (!_isSliding) {
    return;
  }
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onValueChange(CustomSliderEventEmitter::OnValueChange{
        .value = static_cast<Float>(sender.value)
      });
  }
}

- (void)onSlidingStart:(UISlider *)sender
{
  _isSliding = YES;
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onSlidingStart(CustomSliderEventEmitter::OnSlidingStart{
        .value = static_cast<Float>(sender.value)
      });
  }
}

- (void)onSlidingComplete:(UISlider *)sender
{
  _isSliding = NO;
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onSlidingComplete(CustomSliderEventEmitter::OnSlidingComplete{
        .value = static_cast<Float>(sender.value)
      });
  }
}
```

:::info Key Changes
- **EventEmitter**: Type-safe event emission using generated classes
- **Structured events**: Events are C++ structs with defined fields
- **No block properties**: Events are dispatched through EventEmitter
:::

## Step 6: Migrating Commands

Commands allow JavaScript to imperatively control the native view.

### Legacy Implementation

```objc title="ios/YourApp/CustomSliderManager.m"
RCT_EXPORT_METHOD(setValue:(nonnull NSNumber *)reactTag value:(float)value)
{
  [self.bridge.uiManager addUIBlock:^(RCTUIManager *uiManager, NSDictionary<NSNumber *, UIView *> *viewRegistry) {
    CustomSliderView *view = (CustomSliderView *)viewRegistry[reactTag];
    if ([view isKindOfClass:[CustomSliderView class]]) {
      [view setValue:value];
    }
  }];
}

RCT_EXPORT_METHOD(setValueAnimated:(nonnull NSNumber *)reactTag value:(float)value duration:(NSInteger)duration)
{
  [self.bridge.uiManager addUIBlock:^(RCTUIManager *uiManager, NSDictionary<NSNumber *, UIView *> *viewRegistry) {
    CustomSliderView *view = (CustomSliderView *)viewRegistry[reactTag];
    if ([view isKindOfClass:[CustomSliderView class]]) {
      [view setValueAnimated:value duration:duration];
    }
  }];
}
```

### New Architecture Implementation

```objc title="ios/YourApp/CustomSliderComponentView.mm"
#pragma mark - Native Commands

- (void)handleCommand:(const NSString *)commandName args:(const NSArray *)args
{
  RCTCustomSliderHandleCommand(self, commandName, args);
}

- (void)setValue:(float)value
{
  _slider.value = value;
}

- (void)setValueAnimated:(float)value duration:(NSInteger)duration
{
  [UIView animateWithDuration:duration / 1000.0
                   animations:^{
                     self->_slider.value = value;
                   }];
}
```

:::info Key Changes
- **Direct methods**: Commands are implemented as regular methods
- **Generated dispatcher**: RCTCustomSliderHandleCommand handles dispatch
- **Type safety**: Parameters are strongly typed
:::

## Step 7: Complete ComponentView Implementation

Here's the complete ComponentView with all features:

```objc title="ios/YourApp/CustomSliderComponentView.mm"
#import "CustomSliderComponentView.h"

#import <react/renderer/components/CustomSliderNativeComponent/ComponentDescriptors.h>
#import <react/renderer/components/CustomSliderNativeComponent/EventEmitters.h>
#import <react/renderer/components/CustomSliderNativeComponent/Props.h>
#import <react/renderer/components/CustomSliderNativeComponent/RCTComponentViewHelpers.h>

#import <React/RCTConversions.h>
#import "RCTFabricComponentsPlugins.h"

using namespace facebook::react;

@interface CustomSliderComponentView () <RCTCustomSliderViewProtocol>
@end

@implementation CustomSliderComponentView {
  UISlider *_slider;
  BOOL _isSliding;
  Float _step;
}

+ (ComponentDescriptorProvider)componentDescriptorProvider
{
  return concreteComponentDescriptorProvider<CustomSliderComponentDescriptor>();
}

- (instancetype)initWithFrame:(CGRect)frame
{
  if (self = [super initWithFrame:frame]) {
    static const auto defaultProps = std::make_shared<const CustomSliderProps>();
    _props = defaultProps;
    
    _slider = [[UISlider alloc] init];
    _slider.continuous = YES;
    
    [_slider addTarget:self
                action:@selector(onChange:)
      forControlEvents:UIControlEventValueChanged];
    
    [_slider addTarget:self
                action:@selector(onSlidingStart:)
      forControlEvents:UIControlEventTouchDown];
    
    [_slider addTarget:self
                action:@selector(onSlidingComplete:)
      forControlEvents:UIControlEventTouchUpInside | UIControlEventTouchUpOutside];
    
    // Accessibility
    _slider.accessibilityTraits = UIAccessibilityTraitAdjustable;
    
    self.contentView = _slider;
  }
  
  return self;
}

#pragma mark - RCTComponentViewProtocol

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps
{
  const auto &oldViewProps = *std::static_pointer_cast<const CustomSliderProps>(_props);
  const auto &newViewProps = *std::static_pointer_cast<const CustomSliderProps>(props);
  
  if (oldViewProps.value != newViewProps.value) {
    _slider.value = newViewProps.value;
  }
  
  if (oldViewProps.minimumValue != newViewProps.minimumValue) {
    _slider.minimumValue = newViewProps.minimumValue;
  }
  
  if (oldViewProps.maximumValue != newViewProps.maximumValue) {
    _slider.maximumValue = newViewProps.maximumValue;
  }
  
  if (oldViewProps.step != newViewProps.step) {
    _step = newViewProps.step;
  }
  
  // Handle color props
  if (oldViewProps.thumbTintColor != newViewProps.thumbTintColor) {
    _slider.thumbTintColor = RCTUIColorFromSharedColor(newViewProps.thumbTintColor);
  }
  
  if (oldViewProps.minimumTrackTintColor != newViewProps.minimumTrackTintColor) {
    _slider.minimumTrackTintColor = RCTUIColorFromSharedColor(newViewProps.minimumTrackTintColor);
  }
  
  if (oldViewProps.maximumTrackTintColor != newViewProps.maximumTrackTintColor) {
    _slider.maximumTrackTintColor = RCTUIColorFromSharedColor(newViewProps.maximumTrackTintColor);
  }
  
  [super updateProps:props oldProps:oldProps];
}

#pragma mark - Event Handlers

- (void)onChange:(UISlider *)sender
{
  if (!_isSliding) {
    return;
  }
  
  Float value = sender.value;
  
  // Apply step if specified
  if (_step > 0) {
    value = round(value / _step) * _step;
    sender.value = value;
  }
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onValueChange(CustomSliderEventEmitter::OnValueChange{
        .value = value
      });
  }
}

- (void)onSlidingStart:(UISlider *)sender
{
  _isSliding = YES;
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onSlidingStart(CustomSliderEventEmitter::OnSlidingStart{
        .value = static_cast<Float>(sender.value)
      });
  }
}

- (void)onSlidingComplete:(UISlider *)sender
{
  _isSliding = NO;
  
  Float value = sender.value;
  
  // Apply step if specified
  if (_step > 0) {
    value = round(value / _step) * _step;
    sender.value = value;
  }
  
  if (_eventEmitter) {
    std::dynamic_pointer_cast<const CustomSliderEventEmitter>(_eventEmitter)
      ->onSlidingComplete(CustomSliderEventEmitter::OnSlidingComplete{
        .value = value
      });
  }
}

#pragma mark - Native Commands

- (void)handleCommand:(const NSString *)commandName args:(const NSArray *)args
{
  RCTCustomSliderHandleCommand(self, commandName, args);
}

- (void)setValue:(float)value
{
  _slider.value = value;
}

- (void)setValueAnimated:(float)value duration:(NSInteger)duration
{
  [UIView animateWithDuration:duration / 1000.0
                   animations:^{
                     self->_slider.value = value;
                   }];
}

#pragma mark - Accessibility

- (void)updateAccessibilityValue
{
  _slider.accessibilityValue = [NSString stringWithFormat:@"%.1f", _slider.value];
}

@end
```

## JavaScript Wrapper Component

The JavaScript wrapper is the same as the Android version:

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

## Summary

Migrating to the new architecture on iOS involves:

1. **Creating a TypeScript specification** that defines your component's interface (shared with Android)
2. **Creating a ComponentView** that extends RCTViewComponentView
3. **Moving prop handling** from ViewManager to ComponentView
4. **Using EventEmitter** for type-safe event dispatching
5. **Implementing commands** as direct methods with generated dispatch
6. **Leveraging C++ types** for better performance and type safety

### Benefits of Migration

- **Better Performance**: Synchronous layout and rendering
- **Type Safety**: Compile-time checking prevents runtime errors
- **Less Boilerplate**: Code generation reduces manual work
- **Better Memory Management**: More efficient view recycling
- **Cross-Platform Consistency**: Same TypeScript spec for both platforms
- **Future-Proof**: Aligned with React Native's architectural direction

### Key Differences from Android

- **ComponentView pattern**: iOS uses a separate ComponentView class
- **C++ integration**: Direct use of C++ types in Objective-C++
- **View hierarchy**: ContentView pattern for wrapping native views
- **Event handling**: Different syntax but same concept

### Next Steps

- Run Codegen to generate the native interfaces
- Test your migrated component with both architectures
- Update your app's build configuration for the new architecture
- Review the [Fabric Native Components documentation](/docs/fabric-native-components-ios) for advanced features
- Consider implementing more complex features like custom layout

Remember to test thoroughly on both old and new architecture to ensure compatibility during the migration period!