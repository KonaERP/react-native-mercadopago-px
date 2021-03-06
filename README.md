# MercadoPago PX RN [![npm version](https://badge.fury.io/js/%40blackbox-vision%2Freact-native-mercadopago-px.svg)](https://badge.fury.io/js/%40blackbox-vision%2Freact-native-mercadopago-px) [![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT)

RN bridge to integrate `MercadoPago` checkout into a RN app.

## Why this exists?

Currently, `MercadoPago` only brings OSS support for Native SDKs. There's currently not support for RN. Our library uses both SDKs, android and ios, to enable a bridge for calling MercadoPago Checkout from RN land.

We previously developed [react-native-mercadopago-checkout](https://github.com/BlackBoxVision/react-native-mercadopago-checkout). This newer library will suppose a deprecation of the older repository, we don't have plans for maintainance.

## Table of contents

- [Use Case](#use-case)
- [Compatibility](#compatibility)
- [Pre Requisites](#pre-requisites)
- [Installation](#installation)
  - [NPM](#npm)
  - [YARN](#yarn)
- [Library Setup](#library-setup)
  - [IOS](#ios)
    - [Modify AppDelegate.m](#modify-appdelegatem)
    - [Remove Flipper](#remove-flipper)
      - [From AppDelegate.m](#from-appdelegatem)
      - [From Podfile](#from-podfile)
    - [Update Podfile](#update-podfile)
      - [Update IOS Target](#update-ios-target)
      - [Disable Input and Output Paths](#disable-input-and-output-paths)
      - [Modify DoubleConversion, Glog and Folly](#modify-doubleconversion-glog-and-folly)
      - [Add support for Modular Headers](#add-support-for-modular-headers)
      - [Install Pods](#install-pods)
- [Example Usage](#example-usage)
- [Realistic Example](#realistic-example)
- [A Note on Security](#a-note-on-security)
- [API](#api)
  - [Create Payment](#createpayment)
    - [Parameters](#parameters)
    - [Return Value](#return-value)
- [TODOs](#todos)
- [Issues](#issues)
- [Contributing](#contributing)
- [License](#license)

## Use case

You're using RN for building an app, and you need to integrate MercadoPago checkout in your app.

## Compatibility

Our package currently supports apps with **RN >= 0.61.5**. `We don't have a plan currently to support olders ones, but if you need we're open to support it.`

## Pre Requisites

As a pre requisite you'll need the following before integrating the library:

1. A MercadoPago Account
2. A `publicKey` from your MercadoPago Account
3. A `preferenceId` obtained from your servers

If you don't have any of the followings, you can start from here:

1. [Creating a MercadoPago Account](https://www.mercadopago.com.ar/)
2. [Creating a MercadoPago Application](https://applications.mercadopago.com/)
3. [Creating a MercadoPago preference for Checkout Payment](https://www.mercadopago.com.ar/developers/es/reference/preferences/**checkout**preferences/post/)

For Testing Purposes we provide a `cURL` example on how to create a Preference:

```bash
curl -X POST \
    'https://api.mercadopago.com/checkout/preferences?access_token=ACCESS_TOKEN' \
    -H 'Content-Type: application/json' \
    -d '{
      "items": [
        {
          "title": "Dummy Item",
          "description": "Multicolor Item",
          "quantity": 1,
          "currency_id": "ARS",
          "unit_price": 10.0
        }
      ],
      "payer": {
        "email": "payer@email.com"
      }
    }'
```

You'll need to replace `ACCESS_TOKEN` with your application account access token.

If you've more doubts you can read more documentation in this portal:

- [MercadoPago Developers](https://developers.mercadopago.com/)

## Installation

You can install this library via NPM or YARN.

### NPM

```bash
npm i @blackbox-vision/react-native-mercadopago-px
```

### YARN

```bash
yarn add @blackbox-vision/react-native-mercadopago-px
```

## Library Setup

### IOS

Setting up this library is a little bit trickier for `IOS` rathen than `Android`.

#### Modify AppDelegate.m

Replace this line in `AppDelegate.m`:

```objective-c
self.window.rootViewController = rootViewController;
```

For those lines:

```objective-c
UINavigationController *navController = [[UINavigationController alloc] initWithRootViewController:rootViewController];

[navController setToolbarHidden:YES animated:YES];
[navController setNavigationBarHidden:YES];

self.window.rootViewController = navController;
```

#### Remove Flipper

##### From AppDelegate.m

Remove the following lines: 

```objective-c
#if DEBUG
#import <FlipperKit/FlipperClient.h>
#import <FlipperKitLayoutPlugin/FlipperKitLayoutPlugin.h>
#import <FlipperKitUserDefaultsPlugin/FKUserDefaultsPlugin.h>
#import <FlipperKitNetworkPlugin/FlipperKitNetworkPlugin.h>
#import <SKIOSNetworkPlugin/SKIOSNetworkAdapter.h>
#import <FlipperKitReactPlugin/FlipperKitReactPlugin.h>
static void InitializeFlipper(UIApplication *application) {
  FlipperClient *client = [FlipperClient sharedClient];
  SKDescriptorMapper *layoutDescriptorMapper = [[SKDescriptorMapper alloc] initWithDefaults];
  [client addPlugin:[[FlipperKitLayoutPlugin alloc] initWithRootNode:application withDescriptorMapper:layoutDescriptorMapper]];
  [client addPlugin:[[FKUserDefaultsPlugin alloc] initWithSuiteName:nil]];
  [client addPlugin:[FlipperKitReactPlugin new]];
  [client addPlugin:[[FlipperKitNetworkPlugin alloc] initWithNetworkAdapter:[SKIOSNetworkAdapter new]]];
  [client start];
}
#endif
```

```objective-c 
#if DEBUG
  InitializeFlipper(application);
#endif
```

##### From Podfile

Remove the following lines: 

```objective-c
def add_flipper_pods!
  version = '~> 0.33.1'
  pod 'FlipperKit', version, :configuration => 'Debug'
  pod 'FlipperKit/FlipperKitLayoutPlugin', version, :configuration => 'Debug'
  pod 'FlipperKit/SKIOSNetworkPlugin', version, :configuration => 'Debug'
  pod 'FlipperKit/FlipperKitUserDefaultsPlugin', version, :configuration => 'Debug'
  pod 'FlipperKit/FlipperKitReactPlugin', version, :configuration => 'Debug'
end
# Post Install processing for Flipper
def flipper_post_install(installer)
  installer.pods_project.targets.each do |target|
    if target.name == 'YogaKit'
      target.build_configurations.each do |config|
        config.build_settings['SWIFT_VERSION'] = '4.1'
      end
    end
  end
end
```

```objective-c
# Enables Flipper.
#
# Note that if you have use_frameworks! enabled, Flipper will not work and
# you should disable these next few lines.
add_flipper_pods!
post_install do |installer|
  flipper_post_install(installer)
end 
```

#### Update Podfile

##### Update IOS Target

Replace the following line:

```objective-c
platform :ios, '9.0'
```

With the following one:

```objective-c
platform :ios, '10.0'
```

##### Disable Input and Output Paths

After the following line:

```objective-c
platform :ios, '10.0'
```

Attach the following lines:

```objective-c
install! 'cocoapods', :disable_input_output_paths => true
use_frameworks!
```

##### Modify DoubleConversion, Glog and Folly

Replace those lines:

```objective-c
pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'
```

With those ones:

```objective-c
pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec', :modular_headers => false
pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec', :modular_headers => false
pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec', :modular_headers => false
```

##### Add support for Modular Headers

After the following line:

```objective-c
use_native_modules!
```

Attach the following line:

```objective-c
use_modular_headers!
```

##### Install Pods

Install pods by running the following commands:

```bash
cd ios
pod deintegrate
pod install
```

With those steps fully completed, you should be able to build the IOS app accordangly.

## Example Usage

After reading and performing the previous steps, you should be able to import the library and use it like in this example:

```javascript
import Env from 'react-native-config';
import React, { useState } from 'react';
import { StyleSheet, View, Text, TouchableOpacity, Alert } from 'react-native';

import MercadoPagoCheckout from '@blackbox-vision/react-native-mercadopago-px';

import styles from './styles';

// You should create the preference server-side, not client-side but we show client-side for the sake of simplicity
const getPreferenceId = async (payer, ...items) => {
  const response = await fetch(
    `https://api.mercadopago.com/checkout/preferences?access_token=${Env.MP_ACCESS_TOKEN}`,
    {
      method: 'POST',
      body: JSON.stringify({
        items,
        payer: {
          email: payer,
        },
      }),
    }
  );

  const preference = await response.json();

  return preference.id;
};

export default function App() {
  const [paymentResult, setPaymentResult] = useState(null);

  const startCheckout = async () => {
    try {
      const preferenceId = await getPreferenceId('payer@email.com', {
        title: 'Dummy Item Title',
        description: 'Dummy Item Description',
        quantity: 1,
        currency_id: 'ARS',
        unit_price: 10.0,
      });

      const payment = await MercadoPagoCheckout.createPayment({
        publicKey: Env.MP_PUBLIC_KEY,
        preferenceId,
      });

      setPaymentResult(payment);
    } catch (err) {
      Alert.alert('Something went wrong', err.message);
    }
  };

  return (
    <View style={styles.container}>
      <TouchableOpacity onPress={startCheckout}>
        <Text style={styles.text}>Start Payment</Text>
      </TouchableOpacity>
      <Text style={styles.text}>Payment: {JSON.stringify(paymentResult)}</Text>
    </View>
  );
}
```

## Realistic Example

We provide a more real sample app [here](./example).

![Example App Running](./checkout.gif)

## A Note on Security

**For the sake of simplicity in our examples we show the generation of the MercadoPago payment preference from the frontend side. That isn't ideal in the case you want your app to reach a production environment.**

**You should move the preference creation from the frontend to your own backend.**

## API

### createPayment

The function lets you start a MercadoPago Checkout Flow Activity/UI Controller depending on the platform that is running.

#### Parameters

The function receives the following parameters:

- `options`: **[PaymentOptions](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/src/index.tsx#L26)**
  - `publicKey`: **string**
  - `preferenceId`: **string**
  - `advancedOptions`: **[AdvancedOptions](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/src/index.tsx#L7)**
    - `amountRowsEnabled`: **boolean**
    - `bankDealsEnabled`: **boolean**
    - `productId`: **string**
  - `trackingOptions`: **[TrackingOptions](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/src/index.tsx#L3)**
    - `sessionId`: **string**

#### Return Value

The `createPayment` function is async, its return value will be always a `Promise`, but if you unwrap the promise contents you will access the following result object:

- `payment`: **[Payment](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/src/index.tsx#L49)**
  - `id`: **string**
  - `status`: **string**

## Troubleshooting

### It doesn't work with Expo ejected App

Yes. It does! But to be able to work with `Expo`, you need to do the following adjustment in your `Podfile`.

After the following line:

```objective-c
install! 'cocoapods', :disable_input_output_paths => true
```

Attach the following one:

```objective-c
use_frameworks!
```

Then, run the following commands:

```bash
cd ios
pod deintegrate
pod install
```

After this change you should be able to run your Expo ejected app.

**PD: Take care with this, if you use dynamic libraries you'll have some troubles.**

## TODOs

This library is currently in WIP. We've the following pendings:

- [ ] Add support for Color Customization
- [ ] Add support for Font Customization
- [ ] Improve `Payment` object by bringing more properties to JS land

## Issues

Please, open an [issue](https://github.com/BlackBoxVision/react-native-mercadopago-px/issues) following one of the issues templates. We will do our best to fix them.

## Contributing

If you want to contribute to this project see [contributing](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/CONTRIBUTING.md) for more information.

## License

Distributed under the **MIT license**. See [LICENSE](https://github.com/BlackBoxVision/react-native-mercadopago-px/blob/master/LICENSE) for more information.
