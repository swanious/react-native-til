1. 라이브러리 및 의존성 설치

``` 
npm install react-native-camera 
npm install react-native-qrcode-scanner
npm install react-native-permissions
```

2. android일 때, 

   - app/build.gradle

   ```
   android {
       compileSdkVersion rootProject.ext.compileSdkVersion
   
       compileOptions {
       ...
       }
   
       defaultConfig {
           ...
           ...
           // 아래 부분 추가 
           missingDimensionStrategy 'react-native-camera', 'general'
           missingDimensionStrategy 'react-native-camera', 'mlkit'
           // multiDex 에러를 위해 미리 추가해두는 것이 좋음
           multiDexEnabled true
       }
   ```

3. ios일 때,

   - ios/info.plist

   ```
   // 아래 key value 추가
   <key>NSCameraUsageDescription</key>
   <string>Your message to user when the camera is accessed for the first time</string>
   <key>NSMicrophoneUsageDescription</key>
   <string>Your message to user when the microsphone is accessed for the first time</string>
   <key>NSPhotoLibraryUsageDescription</key>
   <string>Your message to user when the photo library is accessed for the first time</string>
   ```

   - ios/podfile 수정

   ```
   target 'ReactNativeBarcode' do
     # Permissions (퍼미션 요청을 위한 코드 추가)
     permissions_path = '../node_modules/react-native-permissions/ios'
     pod 'Permission-Camera', :path => "#{permissions_path}/Camera.podspec"
   
     # Pods for ReactNativeBarcode
     ...
   end
   ```

코드

```
import React, { useState, useRef } from 'react';
import { StyleSheet, View, Text, Linking, TouchableOpacity, ScrollView } from 'react-native';

import { Dimensions } from 'react-native';
const deviceWidth = Dimensions.get('screen').width;
const deviceHeight = Dimensions.get('screen').height;

import QRCodeScanner from 'react-native-qrcode-scanner';

const QRCodeScreen = (props) => {
  const [scan, setScan] = useState(false);
  const [scanResult, setScanResult] = useState(false);
  const [result, setResult] = useState(null);

  const scanner = useRef('');

  const onSuccess = (e) => {
    const check = e.data.substring(0, 4);
    console.log('scanned data' + check);

    setResult(e);
    setScan(false);
    setScanResult(true);

    if (check === 'http') {
      Linking.openURL(e.data).catch((err) => console.error('An error occured', err));
    } else {
      setResult(e);
      setScan(false);
      setScanResult(true);
    }
  };

  const activeQR = () => {
    setScan(true);
  };

  const scanAgain = () => {
    setScan(true);
    setScanResult(false);
  };

  return (
    <View style={styles.container}>
      <View>
        {!scan && !scanResult && (
          <ScrollView>
            <View style={styles.cardView}>
              <TouchableOpacity onPress={activeQR} style={styles.buttonTouchable}>
                <Text style={styles.buttonTextStyle}>Click to Scan !</Text>
              </TouchableOpacity>
            </View>
          </ScrollView>
        )}

        {scanResult && (
          <>
            <Text style={styles.textTitle1}>Result !</Text>
            <View style={scanResult ? styles.scanCardView : styles.cardView}>
              <Text>Type : {result.type}</Text>
              <Text>Result : {result.data}</Text>
              <Text numberOfLines={1}>RawData: {result.rawData}</Text>
              <TouchableOpacity onPress={scanAgain} style={styles.buttonTouchable}>
                <Text style={styles.buttonTextStyle}>Click to Scan again!</Text>
              </TouchableOpacity>
            </View>
          </>
        )}

        {scan && (
          <QRCodeScanner
            reactivate={true}
            showMarker={true}
            ref={(node) => {
              scanner.current = node;
            }}
            onRead={onSuccess}
            topContent={
              <Text style={styles.centerText}>
                Go to <Text style={styles.textBold}>wikipedia.org/wiki/QR_code</Text> on your
                computer and scan the QR code to test.
              </Text>
            }
            bottomContent={
              <View>
                <TouchableOpacity
                  style={styles.buttonTouchable}
                  onPress={() => scanner.current.reactivate()}>
                  <Text style={styles.buttonTextStyle}>OK. Got it!</Text>
                </TouchableOpacity>

                <TouchableOpacity style={styles.buttonTouchable} onPress={() => setScan(false)}>
                  <Text style={styles.buttonTextStyle}>Stop Scan</Text>
                </TouchableOpacity>
              </View>
            }
          />
        )}
      </View>
    </View>
  );
};

export default QRCodeScreen;

const styles = StyleSheet.create({
  container: {
    backgroundColor: 'white',
    height: '100%',
    width: '100%',
  },

  scrollViewStyle: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#99003d',
  },

  textTitle: {
    fontWeight: 'bold',
    fontSize: 18,
    textAlign: 'center',
    padding: 16,
    color: 'white',
  },
  textTitle1: {
    fontWeight: 'bold',
    fontSize: 18,
    textAlign: 'center',
    padding: 16,
    color: 'black',
  },
  cardView: {
    width: deviceWidth - 32,
    height: deviceHeight,
    alignSelf: 'center',
    justifyContent: 'flex-start',
    alignItems: 'center',
    borderWidth: 1,
    borderRadius: 2,
    borderColor: '#ddd',
    borderBottomWidth: 0,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.8,
    shadowRadius: 2,
    elevation: 4,
    marginLeft: 5,
    marginRight: 5,
    backgroundColor: 'white',
  },
  scanCardView: {
    width: deviceWidth - 32,
    height: deviceHeight / 2,
    alignSelf: 'center',
    justifyContent: 'center',
    alignItems: 'center',
    borderWidth: 1,
    borderRadius: 2,
    borderColor: '#ddd',
    borderBottomWidth: 0,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.8,
    shadowRadius: 2,
    elevation: 4,
    marginLeft: 5,
    marginRight: 5,
    marginTop: 10,
    backgroundColor: 'white',
  },
  buttonScan: {
    width: 42,
  },
  descText: {
    padding: 16,
    textAlign: 'justify',
    fontSize: 16,
  },

  highlight: {
    fontWeight: '700',
  },

  centerText: {
    flex: 1,
    fontSize: 18,
    padding: 32,
    color: '#777',
  },
  textBold: {
    fontWeight: '500',
    color: '#000',
  },
  buttonTouchable: {
    fontSize: 21,
    backgroundColor: '#ff0066',
    marginTop: 32,
    borderRadius: 8,
    width: deviceWidth - 62,
    justifyContent: 'center',
    alignItems: 'center',
    height: 44,
  },
  buttonTextStyle: {
    color: 'white',
    fontWeight: 'bold',
  },
});

```

