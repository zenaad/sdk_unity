# UNITY GUIDE &nbsp;<sub>[En](./README.md)</sub>&nbsp;<sub>[Ko](./README.ko.md)</sub>

1. [Overview](#1-overview)

2. [Create an ad in the Zenaad web console and add an Ad ID](#2-create-an-ad-in-the-zenaad-web-console-and-add-an-ad-id)

3. [SDK Installation](#3-sdk-installation)
    * Import
    * Folder Descriptions

4. [Permission](#4-permission)
    * Android
    * iOS

5. [Link](#5-link)
    * Initialize SDK
    * Request AD ready
    * Confirm ad ready
    * Show Ad
    * Position the banner
    * Remove banner
    * Code Sample
    * IListenerZena2d Callback
    * Callback Parameter List
    * Server Response Message List

6. [Test and Approval Request](#6-test-and-approval-request)
    * Test
    * Requesting Approva

7. [Additional Information](#7-additional-information)

<br/><br/><br/><br/><br/>

## 1. overview
---
<br/>

* This document details how to apply SDK to a real project.

* SDK is compatible with Unity 2019.4.5f1 +, android API Level 19 +, and iOS 9.0 +. The ZenaAD Ad ID and latest SDK from the web console must be obtained separately.

* When running the sample project, run after editing in the issued Ad ID.

<br/><br/>

## 2. create an ad in the zenaad web console and add an ad id
---
<br/>

1. Create an App. (App Management/App Creation)

2. Create an Ad. (Ad Management/Ad Creation)

3. Functions can be run within your project using the Ad ID in the generated ad.

<br/><br/>

## 3. sdk installation
---
<br/>

A unity package file containing a sample project and SDK is provided.

* Import

    1. Open Unity Project.

    2. Asset → Import Package → Custom Package → zena2d_sdk.unitypackage → Open

    3. Press the Import button to include the project in the same structure as the file structure set in the package.

* Folder Descriptions

    - Assets\Plugins\Android: Android package used in ZenaAD.
    
    - Assets\Plugins\iOS: IOS package used in ZenaAD.

    - Assets\Zena2d: C# code that provides ZenaAD Ad calling API.

    - Assets\Zena2d_Guide: Sample project and executable codes.

<br/><br/>

## 4. permission
---
<br/>

* Android

    -  If there is another AndroidManifest.xml file in the project and the AndroidManifest.xml file is included in the .aar package of the ZenaAD, the SDK will be automatically merged at build time.   
    Permission currently being used in ZenaAD:
    ```
    <uses-permission android:name="android.permission.INTERNET" />
    ```

    - If ads are not visible after applying Proguard:   
    If you have checked User Proguard File in Publishing Setting, Assets\Plugins\Android\proguard-user.txt should exist. Add the following here.
    ```
    -keep class com.rhaon.aos_zena2d_sdk.**
    -keep interface com.rhaon.aos_zena2d_sdk.**
    ```

* iOS

    -  After building to allow iOS network use, add and change the following key values in the info.plist file in the generated xcode project.

       ![ios_plist1](Image/ios_plist1.JPG "info.plist")

    -  In iOS 14 or later, the following key values are required in the info.plist file due to IDFA policy change.   
    (String : The app is trying to access IDFA for advertising purposes.)

       ![ios_plist2](Image/ios_plist2.JPG "info.plist")

<br/><br/>

## 5. link
---
<br/>

* Initialize SDK
    ```c#
    public void Client.Init( "BANNER ID", "INTERSTITIAL ID", "VIDEO ID" );
    ```
<br/>

* Request AD ready
    ```c#
    public void Client.ReadyAd( Product, "BANNER ID or INTERSTITIAL ID or VIDEO ID" );
    // Product : Product.BANNER, Product.INTERSTITIAL, Product.VIDEO
    // (Ad types : Banner Ad, Full-screen Ad, Video Ad)
    ```
<br/>

* Confirm ad ready
    ```c#
    public bool Client.IsReadyAd( Product );
    // Product : Product.BANNER, Product.INTERSTITIAL, Product.VIDEO
    // (Ad types : Banner Ad, Full-screen Ad, Video Ad)
    ```
<br/>

* Show Ad
    ```c#
    public void Client.CreateAd( Product );
    // Product : Product.BANNER, Product.INTERSTITIAL, Product.VIDEO
    // (Ad types : Banner Ad, Full-screen Ad, Video Ad)
    ```
<br/>

* Position the banner
    ```c#
    public void Client.SetBannerGravity( BannerMode );
    // BannerMode : BannerMode.BOTTOM, BannerMode.TOP, BannerMode.SOFT_KEY
    // (Banner Position : Bottom, Top, Softkey_Top)
    ```
<br/>

* Remove banner
    ```c#
    public void Client.RemoveBanner( );
    ```
<br/>

* Code Sample
    ```c#
    private Client client;
    
    void Start ( ) {
        client = Zena2dClient.BuildClient( );
        client.setBannerGravity(BannerMode.BOTTOM);
        client.callbackInit = OnInit;
        client.callbackError = OnError;
        client.callbackReady = OnReady;
        client.callbackCreate = OnCreate;
        client.callbackReward = OnReward;
        client.callbackClick = OnClick;
        client.callbackClose = OnClose;
        client.Init( "BANNER ID", "INTERSTITIAL ID", "VIDEO ID" );
    }
    
    //--------------------callback--------------------//
    
    public void OnInit( bool isInit, string message ) {
        Debug.Log( "OnInit : " + isInit + " : " + message);
        //ex : Banner Ad Ready
        if ( isInit ) client.ReadyAd( Product.BANNER, "BANNER ID" );
    }

    public void OnError( int product, string message, string detail ) {
        Debug.Log( "OnError : " + product + " : " + message + " : " + detail);
    }
    
    public void OnReady( int product, bool success, string message ) {
        Debug.Log( "OnReady : " + product + " : " + success + " : " + message );
        //ex : Banner Ad creation and check ready
        if ( client.IsReadyAd( product )) client.CreateAd( product );
    }
    
    public void OnCreate( int product, bool success, string message ) {
        Debug.Log( "OnCreate : " + product + " : " + success + " : " + message );
    }
    
    public void OnReward( int product, string reward, int count ) {
        Debug.Log( "OnReward : " + product + " : " + reward + " : " + count );
    }
    
    public void OnClick( int product ) {
        Debug.Log( "OnClick : " + product );
    }
    
    public void OnClose( int product, string medID ) {
        Debug.Log( "OnClose : " + product + " : " + medID );
        //ex : Close ad while preparing for next ad
        client.ReadyAd( (Product)product, medID );
    }
    ```
<br/>

* IListenerZena2d Callback

    |Callback|Description|
    |---|---|
    |callbackInit(bool, string) |client.Init( ); -> Initializes SDK and indicates if successful.|
    |callbackError(int, string, string) |Called on network errors and response errors.|
    |callbackReady(int, bool, string) |client.ReadyAd( ); -> Processes an ad request, loads the ad, and indicates if the ad is ready.|
    |callbackCreate(int, bool, string) |client.CreateAd( ); -> Displays prepared ad.|
    |callbackReward(int, string, int) |Called when the reward conditions for compensating ads are met.|
    |callbackClick(int) |Called when an ad is clicked.|
    |callbackClose(int, string) |Invoked when the ad and point windows are closed.|
<br/>

* Callback Parameter List

    |Parameter|Description|
    |---|---|
    |bool isInit |Ad ready status.|
    |bool success |Function success/failure.|
    |string message |Response status and messages.|
    |string detail |Detailed error message.|
    |string reward |Name of the reward entered for a compensating ad.|
    |string medID |The medID sent when preparing an ad.|
    |int product |Ad Category - 0 or 1 or 2 (0 : BANNER, 1 : INTERSTITIAL, 2 : VIDEO)|
    |int count |The reward count set for a compensating ad.|
<br/>

* Server Response Message List

    |Message|Description|
    |---|---|
    |SUCCESS |Success.|
    |UNKNOWN |Unknown Error.|
    |UNKNOWN_MEDID |Unknown Media (Ad) ID.|
    |AD_DISABLED |Ad is currently disabled. (If broadcast is stopped in the Web Console Ad Management)|
    |AD_NOTFOUND |There is no ad.|
    |AD_EXPIRED |The prepared Ad has expired. ReadyAd( ) needed.|
    |ALREADY_ADREQ |Duplicate Ad request. – sdk error.|
    |EXCEEDED_IMPRESSION |Exposure limit exceeded. (If exposure count is set in the Web Console Ad Management)|

<br/><br/>

## 6. test and approval request
---
<br/>

* Test

    - If the following ZenaAD native ads are displayed, then the system is running normally.

        |Type|Example|
        |:---:|:---:|
        |Banner |<img src="Image/banner.jpg" width="320px" height="50px" title="Banner Image" alt="banner"></img>|
        |Full-screen |<img src="Image/front.jpg" width="240px" height="360px" title="Full Image" alt="interstitial"></img>|
        |Video |<img src="Image/video.jpg" width="384px" height="216px" title="Video Image" alt="videoImage"></img>|
<br/>

* Requesting Approval

    - Requesting approval immediately before or after launch will broadcast your ad and accumulate balance.

    - Here is an example of an approval request email:   
    <br/>
    Contents : help@zenaad.com <br/>
    Company Info : ZenaAD account email (required): ex) publisher@zenaad.com <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Company Name (required): ex) ZenaAD Co., Ltd. <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; App Name (required):  A searchable name in the Store, if <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Store Address (optional): Valid URL Address

<br/><br/>

## 7. additional information
---
<br/>

*  ZenaAD uses ad identifiers (ADID, IDFA) and saves cash data for efficient resource use. (Up to 256 MB)

<br/><br/>
