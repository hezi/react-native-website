---
id: native-modules-migration-ios
title: Native Modules Migration - iOS
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Native Modules Migration - iOS

This guide will walk you through migrating your existing iOS native modules from the legacy architecture to the new architecture (Turbo Modules). We'll use a `CalendarModule` to demonstrate the migration process.

## Introduction

The new architecture on iOS brings the same benefits as Android: improved performance through lazy loading, better type safety with code generation, and direct communication between JavaScript and native code. This guide shows you how to migrate your existing iOS native modules step by step.

## Prerequisites

Before starting the migration, ensure you have:

1. A React Native app with the new architecture enabled
2. Basic knowledge of iOS development (Objective-C or Swift)
3. An existing native module that you want to migrate
4. TypeScript or Flow set up in your project

## Project Structure Overview

Here's how the structure changes between legacy and new architecture:

### Legacy Architecture
```
ios/YourApp/
├── CalendarModule.h
├── CalendarModule.m
└── YourApp-Bridging-Header.h (if using Swift)
```

### New Architecture
```
ios/YourApp/
├── CalendarModule.h
├── CalendarModule.mm  # Note the .mm extension for Objective-C++
└── YourApp-Bridging-Header.h (if using Swift)

js/
└── NativeCalendarModule.ts  # TypeScript specification (same as Android)
```

## Step 1: Creating the TypeScript Specification

The TypeScript specification is identical to the Android version. This is one of the benefits of the new architecture - you write the spec once and use it for both platforms.

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

```objc title="ios/YourApp/CalendarModule.h"
#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>

@end
```

```objc title="ios/YourApp/CalendarModule.m"
#import "CalendarModule.h"

@implementation CalendarModule

RCT_EXPORT_MODULE()

// Module implementation continues...

@end
```

### New Architecture

```objc title="ios/YourApp/CalendarModule.h"
#import <NativeCalendarModuleSpec/NativeCalendarModuleSpec.h>

NS_ASSUME_NONNULL_BEGIN

@interface CalendarModule : NSObject <NativeCalendarModuleSpec>

@end

NS_ASSUME_NONNULL_END
```

```objc title="ios/YourApp/CalendarModule.mm"
#import "CalendarModule.h"
#import <React/RCTConversions.h>

@interface CalendarModule () <RCTEventEmitter>
@end

@implementation CalendarModule {
  NSInteger _listenerCount;
}

RCT_EXPORT_MODULE()

// Module implementation continues...

@end
```

:::info Key Changes
- **Header file**: Now imports the generated spec header instead of `RCTBridgeModule`
- **Implementation file**: Changed from `.m` to `.mm` for Objective-C++ support
- **Protocol conformance**: Conforms to generated `NativeCalendarModuleSpec` protocol
- **Base class**: Changed from `RCTEventEmitter` to `NSObject` with protocol conformance
:::

## Step 3: Migrating Synchronous Methods

Synchronous methods require minimal changes in the new architecture.

### Legacy Implementation

```objc title="ios/YourApp/CalendarModule.m"
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getDefaultCalendarName)
{
  // Get the default calendar name
  return @"Personal Calendar";
}
```

### New Architecture Implementation

```objc title="ios/YourApp/CalendarModule.mm"
- (NSString *)getDefaultCalendarName
{
  // Get the default calendar name
  return @"Personal Calendar";
}
```

### JavaScript Usage

```javascript
import NativeCalendarModule from './NativeCalendarModule';

const calendarName = NativeCalendarModule.getDefaultCalendarName();
console.log('Default calendar:', calendarName);
```

:::info Key Changes
- **No RCT_EXPORT macros**: Methods are defined in the TypeScript spec
- **Standard Objective-C methods**: Implement methods as regular instance methods
- **Return types must match**: The spec ensures type safety
:::

## Step 4: Migrating Callbacks and Promises

### Callback-based Methods

#### Legacy Implementation

```objc title="ios/YourApp/CalendarModule.m"
RCT_EXPORT_METHOD(getCalendarEvents:(NSString *)startDate
                            endDate:(NSString *)endDate
                           callback:(RCTResponseSenderBlock)callback)
{
  @try {
    NSMutableArray *events = [NSMutableArray new];
    
    // Simulate fetching calendar events
    NSDictionary *event1 = @{
      @"id": @"1",
      @"title": @"Team Meeting",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event1];
    
    NSDictionary *event2 = @{
      @"id": @"2",
      @"title": @"Project Review",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event2];
    
    callback(@[[NSNull null], events]);
  } @catch (NSException *exception) {
    callback(@[exception.reason, [NSNull null]]);
  }
}
```

#### New Architecture Implementation

```objc title="ios/YourApp/CalendarModule.mm"
- (void)getCalendarEvents:(NSString *)startDate
                  endDate:(NSString *)endDate
                 callback:(RCTResponseSenderBlock)callback
{
  @try {
    NSMutableArray *events = [NSMutableArray new];
    
    // Simulate fetching calendar events
    NSDictionary *event1 = @{
      @"id": @"1",
      @"title": @"Team Meeting",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event1];
    
    NSDictionary *event2 = @{
      @"id": @"2",
      @"title": @"Project Review",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event2];
    
    callback(@[[NSNull null], events]);
  } @catch (NSException *exception) {
    callback(@[exception.reason, [NSNull null]]);
  }
}
```

### Promise-based Methods

#### Legacy Implementation

```objc title="ios/YourApp/CalendarModule.m"
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                            startDate:(NSString *)startDate
                              endDate:(NSString *)endDate
                             location:(NSString *)location
                             resolver:(RCTPromiseResolveBlock)resolve
                             rejecter:(RCTPromiseRejectBlock)reject)
{
  @try {
    // Simulate creating a calendar event
    NSString *eventId = [NSString stringWithFormat:@"event_%lld", 
                        (long long)([[NSDate date] timeIntervalSince1970] * 1000)];
    
    // In a real implementation, you would create the event here
    // For this example, we'll just return the generated ID
    
    resolve(eventId);
  } @catch (NSException *exception) {
    reject(@"CREATE_EVENT_ERROR", @"Failed to create calendar event", nil);
  }
}
```

#### New Architecture Implementation

```objc title="ios/YourApp/CalendarModule.mm"
- (void)createCalendarEvent:(NSString *)title
                  startDate:(NSString *)startDate
                    endDate:(NSString *)endDate
                   location:(NSString * _Nullable)location
                    resolve:(RCTPromiseResolveBlock)resolve
                     reject:(RCTPromiseRejectBlock)reject
{
  @try {
    // Simulate creating a calendar event
    NSString *eventId = [NSString stringWithFormat:@"event_%lld", 
                        (long long)([[NSDate date] timeIntervalSince1970] * 1000)];
    
    // In a real implementation, you would create the event here
    // For this example, we'll just return the generated ID
    
    resolve(eventId);
  } @catch (NSException *exception) {
    reject(@"CREATE_EVENT_ERROR", @"Failed to create calendar event", nil);
  }
}
```

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
- **No RCT_EXPORT_METHOD macro**: Methods are standard Objective-C methods
- **Method signatures**: Must match the TypeScript spec exactly
- **Nullable parameters**: Use `_Nullable` for optional parameters
:::

## Step 5: Migrating Constants

Constants handling changes slightly in the new architecture.

### Legacy Implementation

```objc title="ios/YourApp/CalendarModule.m"
- (NSDictionary *)constantsToExport
{
  return @{
    @"DEFAULT_EVENT_STATUS": @"CONFIRMED",
    @"CALENDAR_PERMISSIONS": @{
      @"READ": @"NSCalendarsUsageDescription",
      @"WRITE": @"NSCalendarsUsageDescription"
    }
  };
}

+ (BOOL)requiresMainQueueSetup
{
  return NO;
}
```

### New Architecture Implementation

```objc title="ios/YourApp/CalendarModule.mm"
- (facebook::react::ModuleConstants<JS::NativeCalendarModule::Constants::Builder>)getConstants
{
  return [self constantsToExport];
}

- (facebook::react::ModuleConstants<JS::NativeCalendarModule::Constants::Builder>)constantsToExport
{
  return facebook::react::typedConstants<JS::NativeCalendarModule::Constants::Builder>({
    .DEFAULT_EVENT_STATUS = @"CONFIRMED",
    .CALENDAR_PERMISSIONS = @{
      @"READ": @"NSCalendarsUsageDescription",
      @"WRITE": @"NSCalendarsUsageDescription"
    }
  });
}

+ (BOOL)requiresMainQueueSetup
{
  return NO;
}
```

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
- **Type-safe constants**: Using C++ typed constants for better type safety
- **Builder pattern**: Constants use a builder pattern in the new architecture
:::

## Step 6: Migrating Event Emitters

Event emission requires implementing event listener methods and proper event management.

### Legacy Implementation

```objc title="ios/YourApp/CalendarModule.m"
- (NSArray<NSString *> *)supportedEvents
{
  return @[@"calendarEventReminder"];
}

- (void)sendEventWithName:(NSString *)eventName body:(id)body
{
  [self sendEventWithName:eventName body:body];
}

RCT_EXPORT_METHOD(startCalendarEventReminders)
{
  // Simulate sending reminders
  NSDictionary *reminder = @{
    @"eventId": @"event_123",
    @"title": @"Team Meeting starting in 15 minutes",
    @"timestamp": @([[NSDate date] timeIntervalSince1970] * 1000)
  };
  
  [self sendEventWithName:@"calendarEventReminder" body:reminder];
}
```

### New Architecture Implementation

```objc title="ios/YourApp/CalendarModule.mm"
@implementation CalendarModule {
  NSInteger _listenerCount;
}

- (NSArray<NSString *> *)supportedEvents
{
  return @[@"calendarEventReminder"];
}

- (void)addListener:(NSString *)eventName
{
  _listenerCount += 1;
}

- (void)removeListeners:(double)count
{
  _listenerCount -= (NSInteger)count;
  if (_listenerCount < 0) {
    _listenerCount = 0;
  }
}

- (void)sendEventIfListening:(NSString *)eventName body:(id)body
{
  if (_listenerCount > 0) {
    [self sendEventWithName:eventName body:body];
  }
}

- (void)startCalendarEventReminders
{
  // Simulate sending reminders
  NSDictionary *reminder = @{
    @"eventId": @"event_123",
    @"title": @"Team Meeting starting in 15 minutes",
    @"timestamp": @([[NSDate date] timeIntervalSince1970] * 1000)
  };
  
  [self sendEventIfListening:@"calendarEventReminder" body:reminder];
}
```

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
- **Conditional emission**: Only emit events when there are active listeners
:::

## Complete Example

Here's the complete migrated module:

```objc title="ios/YourApp/CalendarModule.h"
#import <NativeCalendarModuleSpec/NativeCalendarModuleSpec.h>

NS_ASSUME_NONNULL_BEGIN

@interface CalendarModule : NSObject <NativeCalendarModuleSpec>

@end

NS_ASSUME_NONNULL_END
```

```objc title="ios/YourApp/CalendarModule.mm"
#import "CalendarModule.h"
#import <React/RCTConversions.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule () <RCTEventEmitter>
@end

@implementation CalendarModule {
  NSInteger _listenerCount;
}

RCT_EXPORT_MODULE()

- (NSString *)getDefaultCalendarName
{
  return @"Personal Calendar";
}

- (void)getCalendarEvents:(NSString *)startDate
                  endDate:(NSString *)endDate
                 callback:(RCTResponseSenderBlock)callback
{
  @try {
    NSMutableArray *events = [NSMutableArray new];
    
    NSDictionary *event1 = @{
      @"id": @"1",
      @"title": @"Team Meeting",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event1];
    
    NSDictionary *event2 = @{
      @"id": @"2",
      @"title": @"Project Review",
      @"startDate": startDate,
      @"endDate": endDate
    };
    [events addObject:event2];
    
    callback(@[[NSNull null], events]);
  } @catch (NSException *exception) {
    callback(@[exception.reason, [NSNull null]]);
  }
}

- (void)createCalendarEvent:(NSString *)title
                  startDate:(NSString *)startDate
                    endDate:(NSString *)endDate
                   location:(NSString * _Nullable)location
                    resolve:(RCTPromiseResolveBlock)resolve
                     reject:(RCTPromiseRejectBlock)reject
{
  @try {
    NSString *eventId = [NSString stringWithFormat:@"event_%lld", 
                        (long long)([[NSDate date] timeIntervalSince1970] * 1000)];
    resolve(eventId);
  } @catch (NSException *exception) {
    reject(@"CREATE_EVENT_ERROR", @"Failed to create calendar event", nil);
  }
}

- (NSDictionary * _Nullable)getCalendarById:(NSString *)calendarId
{
  if ([calendarId isEqualToString:@"primary"]) {
    return @{
      @"id": @"primary",
      @"title": @"Personal Calendar",
      @"isPrimary": @(YES)
    };
  }
  return nil;
}

- (facebook::react::ModuleConstants<JS::NativeCalendarModule::Constants::Builder>)getConstants
{
  return [self constantsToExport];
}

- (facebook::react::ModuleConstants<JS::NativeCalendarModule::Constants::Builder>)constantsToExport
{
  return facebook::react::typedConstants<JS::NativeCalendarModule::Constants::Builder>({
    .DEFAULT_EVENT_STATUS = @"CONFIRMED",
    .CALENDAR_PERMISSIONS = @{
      @"READ": @"NSCalendarsUsageDescription",
      @"WRITE": @"NSCalendarsUsageDescription"
    }
  });
}

- (NSArray<NSString *> *)supportedEvents
{
  return @[@"calendarEventReminder"];
}

- (void)addListener:(NSString *)eventName
{
  _listenerCount += 1;
}

- (void)removeListeners:(double)count
{
  _listenerCount -= (NSInteger)count;
  if (_listenerCount < 0) {
    _listenerCount = 0;
  }
}

- (void)sendEventIfListening:(NSString *)eventName body:(id)body
{
  if (_listenerCount > 0) {
    [self sendEventWithName:eventName body:body];
  }
}

- (void)startCalendarEventReminders
{
  NSDictionary *reminder = @{
    @"eventId": @"event_123",
    @"title": @"Team Meeting starting in 15 minutes",
    @"timestamp": @([[NSDate date] timeIntervalSince1970] * 1000)
  };
  
  [self sendEventIfListening:@"calendarEventReminder" body:reminder];
}

+ (BOOL)requiresMainQueueSetup
{
  return NO;
}

@end
```

## Summary

Migrating to the new architecture on iOS involves:

1. **Creating a TypeScript specification** that defines your module's interface (shared with Android)
2. **Changing file extension** from `.m` to `.mm` for Objective-C++ support
3. **Conforming to generated protocol** instead of `RCTBridgeModule`
4. **Removing RCT_EXPORT macros** as methods are defined in the spec
5. **Implementing listener methods** for event emitters
6. **Using typed constants** for better type safety

### Benefits of Migration

- **Better Performance**: Direct communication between JavaScript and native code
- **Type Safety**: Compile-time checking prevents runtime errors
- **Code Generation**: Less boilerplate code to maintain
- **Cross-Platform Consistency**: Same TypeScript spec for both platforms
- **Future-Proof**: Aligned with React Native's architectural direction

### Next Steps

- Test your migrated module with both architectures enabled
- Consider migrating your Native Components next
- Update your app's build configuration for the new architecture
- Review the [Turbo Native Modules documentation](/docs/turbo-native-modules-ios) for advanced features

Remember to test thoroughly on both old and new architecture to ensure compatibility during the migration period!