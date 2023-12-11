# Sử dụng Expo CLI để khởi tạo react native app

### CLI: giao diện dòng lệnh

- expo CLI: dễ dàng hơn, nhiều tiền lợi hơn, tiếp cận camera, ... dễ hơn
- react CLI: làm việc với các java, objectC, .. dễ hơn

### Cài đặt expo CLI: `npm install -g expo-cli`

Tạo project: `expo init RNApplication`

- 4 lựa chọn :
  - 1 app trống
  - 1 app trống nhưng sử dụng typescript
  - 1 app đã thêm điều hướng với react-navigation và typescript
  - 1 app có 1 số hỗ trợ tối thiểu, không sử dụng quá nhiều expo

### Expo CLI hỗ trợ phần mềm "Expo Go" trên các thiết bị di động để demo ứng dụng

- Khởi chạy ứng với Expo Go:
  - terminal: `npm start`
  - sau khi build, trên terminal xuất hiện QR code, có thể sử dụng Expo Go của thiết bị android quét QR code
  - sau khi build cũng khởi chạy 1 tab browser chứa QR code, sử dụng camera của iphone quét QR code để mở ra Expo Go

### Sử dụng Android Studio để thay thế thiết bị android

- Chọn More Actions
- Chọn Virtual Divice Manager
- Create Device hoặc sử dụng device có sẵn. Nên chọn các device có sử dụng CH Play trong cột Play Store để Expo có thể truy cập được thiết bị máy ảo
- nhấn 'a' trên terminal để mở ứng dụng trên máy ảo

# Basics

### Các built into component có sẵn của react-native

- TextInput ~ input : dùng để nhập văn bản
- View ~ div : dùng để chứa và bố trí các component
- Text ~ h1, h2, h3 , ... : dùng để chứa nội dung văn bản
- Image: dùng hiển thị ảnh, cung cấp prop: source{require('local_path_image')} hoặc source={{uri: url_image}}
- Button : component tự đóng, sử dụng prop 'title' để viết nội dung
  ....

### Không có css, sử dụng "Inline Styles" hoặc "StyleSheet" được viết bằng JS

- Sử dụng prop 'style'

```js
/*inline style*/
<Text style={{ color: 'gray', fontWeight: '400', fontSize: 14 }}></Text>
```

- Sử dụng StyleSheet object thay cho inline style

```js
import { StyleSheet } from 'react-native',
/*function component*/
const CustomComponent = () => {
return (<Text style={styles.text}></Text>);
}
```

```js
/*tạo object styles ngoài component*/
const styles = StyleSheet.create({
  text: { color: 'gray', fontWeight: '400', fontSize: 14 },
});
```

- Sự khác biệt style các component giữa 2 nền tảng IOS và Android, có thể bao bọc các component bằng component `View` hoặc viết mã riêg cho từng nền tảng

- Style cho các component không phân tầng, không có sự kế thừa thuộc tính từ component cha xuống con

- `ScrollView` component giúp screen có thể trượt xem những nội dung vượt quá kích cỡ của view. không gian `ScrollView` được giới hạn với 1 component container chứa nó => thường dùng `View` với style về `height` để giới hạn `ScrollView`. `ScrollView` chỉ phù hợp với những nội dung ngắn, cố định độ dài như 1 bài báo, không phù hợp với hiển thị danh sách động nội dung cực kỳ dài tránh vấn đề về hiệu suất

- `FlatList` tương tự `ScrollView` nhưng chỉ hiển thị nội dung chứa trong màn hình, phần còn lại sẽ lazy load khi người dùng cuộn đến. `FlatList` cung cấp 1 ngưỡng tải trước nội dung để hiển thị sẵn

- Sử dụng `FlatList` để hiển thị 1 danh sách, không còn sử dụng `map()` nữa. prop `data` chứa danh sách data cần hiển thị, prop `renderItem` yêu cầu 1 function nhận từng `item` của `data` return về JSX code cho từng phần tử muốn hiển thị cả danh sách. `FlatList` mặc định có chiều dài vô hạn, để `FlatList` có thể scroll cần bọc nó trong 1 `View`, giới hạn ngưỡng chiều dài tối đa cho `FlatList`

- Cung cấp nhiều style cho 1 component bằng cách cung cấp prop `style` 1 mảng các style

```js
// style ở phía sau trong mảng sẽ ghi đè style phí trước nếu có thuộc tinhd trùng nhau
style={[styles.buttonInnerContainer, styles.pressed, { opacity: 0.5}]}
```

```js
/* có 2 cách cung cấp `key` cho từng item trong list:
 * - thêm thuộc tính `key` cho mỗi object phần tử của list
 * - thêm prop `keyExtractor` cho component `Flatlist`, prop yêu cầu 1 funtion nhận vào `item` và `index`*/
<FlatList
  data={list}
  renderItem={(item) => {
    /* object item chứa các thuộc tính:
     * - index: chỉ mục của item trong list
     * - ỉtem: giá trị item trong list */
    return (
      <View>
        <Text>{item.item}</Text>
      </View>
    );
  }}
  keyExtractor={(item, index) => {
    return index;
  }}
  /*numColumns={2} : 2 item 1 cột, mặc định là 1/
/>
```

- Component `Pressale` (`Touchable` ở react-native cũ) là component wrapper bao bọc các component, khiến nó có thêm sự kiện `onPress`

```js
/*sử dụng onPress để gọi đến 1 function của component cha*/
const ListComponent = () => {
  const [list, setList] = useState([
    { id: 1, text: 'A' },
    { id: 2, text: 'B' },
    { id: 3, text: 'C' },
  ]);

  function deleteItemHandler(id) {
    setList((prev) => prev.filter((item) => item.id !== id));
  }

  return (
    <FlatList
      data={list}
      renderItem={(item) => <ItemComponent text={item.item.text} id={item.item.id} onDeleteItem={deleteItemHandler} />}
      keyExtractor={(item, index) => {
        return index;
      }}
    />
  );
};

const ItemComponent = (props) => {
  // sử dụng bind() để gắn logic của onDeleteItem với onPress
  return (
    <Pressable onPress={props.onDeleteItem.bind(this, props.id)}>
      {/*<Pressable onPress={() => props.onDeleteItem(props.id)}>*/}
      <View>
        <Text>{props.text}</Text>
      </View>
    </Pressable>
  );
};
```

- style cho 1 số feedback của `Pressable`

```jsx
// android_ripple: thuộc tính thay đổi style khi nhấn vào của android
// style: style cho component, cũng nhận function thay thế để kiểm tra từng sự kiện, với `pressed` kiểm tra sự kiện ấn vào
<Pressable
  android_ripple={{ color: '#210644' }}
  style={({ pressed }) => pressed && { opacity: 0.5 }}
  onPress={props.onDeleteItem.bind(this, props.id)}
>
  <View>
    <Text>{props.text}</Text>
  </View>
</Pressable>
```

- Component `Modal` hiển thị modal mặc định phủ lên trên toàn bộ nội dung của screen, chiếm toàn bộ screen, `Modal` gồm các prop:
  - visible = true | false
  - animationType = fade | none | slide
- Component `Image` hiển thị hình ảnh, sử dụng prop `source` để chỉ định đường dẫn ảnh

```js
<Image source={require('../assets/images/a.png')} />
```

- Component `StatusBar` import từ `expo-status-bar`, giúp thay đổi thanh status bên trên screen, sử dụng prop `style` để thay đổi giao diện status bar, mặc định `style` có các giá trị "dark | auto | light | inverted"

# Custom 1 button:

```jsx
import { View, Text, Pressable, StyleSheet } from 'react-native';

import Colors from '../../constants/colors';

function PrimaryButton({ children, onPress }) {
  return (
    <View style={styles.buttonOuterContainer}>
      <Pressable
        /*check xem có pressed thì thêm style styles.pressed cho button*/
        style={({ pressed }) => (pressed ? [styles.buttonInnerContainer, styles.pressed] : styles.buttonInnerContainer)}
        onPress={onPress}
        /*style khi pressed riêng của android*/
        android_ripple={{ color: Colors.primary600 }}
      >
        <Text style={styles.buttonText}>{children}</Text>
      </Pressable>
    </View>
  );
}

export default PrimaryButton;

const styles = StyleSheet.create({
  buttonOuterContainer: {
    borderRadius: 28,
    margin: 4,
    overflow: 'hidden',
  },
  buttonInnerContainer: {
    backgroundColor: Colors.primary500,
    paddingVertical: 8,
    paddingHorizontal: 16,
    elevation: 2,
  },
  buttonText: {
    color: 'white',
    textAlign: 'center',
  },
  pressed: {
    opacity: 0.75,
  },
});
```

- Component `SafeAreaView` bao quanh phần content hiển thị tránh các ví trí tai thỏ trên iphone hay giọt nước trên android

# Một số thuộc tính style khác so với trên web

- `boxShadow` :
  -thay bằng `elevation` trên android
  - thay bằng `shadowColor`, `shadowOffset`, `shadowOpacity`và `shadowRadious` cho ios, kết hợp `backgroundColor` để hiển thị rõ shadow cho ios,
    > lưu ý thuộc tính `overflow` là `hidden` thì sẽ không hiển thi bóng trên ios => cần lấy 1 `View` bọc quanh nội dung của component cần đổ bóng và để nó với thuộc tính `hidden` thay thế cho việc component cần đổ bóng phải đặt thuộc tính `hidden`

# Alert

- giúp hiển thị cảnh báo

```js
Alert.alert(): tạo ra cảnh báo có nút
Alert.prompt(): tạo ra cảnh báo nhập giá trị
```

```js
Alert.alert('Invalid number!', 'Number has to be a number between 1 and 99', [
  { text: 'Okay', style: 'destructive', onPress: () => setEnteredNumber('') },
]);
```

# Responsive UIs

- Sử dụng các thuộc tính `maxWidth` hay `minWidth` để style, sử dụng các giá trị `50%`, `80%` thay cho các số cụ thể
- Sử dụng `Dimensions` từ 'react-native', sử dụng bên ngoài component kết hợp với `StyleSheet`

```js
import { Dimensions } from 'react-native';

// lấy ra Demensions bên ngoài component
const deviceWidth = Demensions.get('window').width;
// Demensions.get('screen | window'), với IOS window tương tự như screen,; với Android screen là toàn bộ chiều rồng và cao bao gồm cả thanh trạng thái, window thì loại trừ thanh trạng thái

// Sử dụng deviceWidth cùng các thuộc tính style
{
  padding: deviceWidth < 380 ? 12 : 23,
}
```

- Hướng khoá màn hình mặc định là hướng dọc, đặt ở app.json với thuộc tính `orientation` bằng `portrait`. Đổi giá trị của `orientation` thành `default` sẽ cho phép màn hình có thể xoay. Lưu ý nếu cho phép màn hình xoay ngang, cần style cho component theo `Demensions.get('window').height`

- `Demensions` lấy ra ngoài component chỉ được tính giá trị 1 lần duy nhất khi lần đầu render component, khi kết hợp với `orientation`, giá trị tính toán từ `Demensions` để đưa vào style không bị thay đổi khi xoay màn hình => cần tính toán động giá trị height width

- `useWindowDimensions()` là hooks cung cấp giá trị height và width động tuỳ thuộc vào việc xoay màn hình. Kết hợp với việc viết style inline giúp thay đổi các giá trị style động

```js
import { useWindowDimensions } from 'react-native';

// sử dụng trong component
const { width, height } = useWindowDimensions();
```

- Toggle hiển thị bàn phím có thể che mất 1 phần screen, sử dụng component `KeyboardAvoidingView` để bọc nội dung screen, khi hiển thị bàn phím, phần screen bị che lấp sẽ được dịch lên trên bàn phím. prop `style` cung cấp style cho `KeyboardAvoidingView`, prop `behavior` cung cấp hành vi cho `KeyboardAvoidingView` (nên sử dụng `behavior` với giá trị là `position`). Kết hợp với `ScrollView` được bọc lấy `KeyboardAvoidingView` để nội dung che đi có thể hiển thị đầy đủ

- `Platform` lấy ra được các giá trị của nên tảng đang sử dụng, có thể dựa vào đó thay đổi style

```js
// các giá trị nhận được là and  roid, ios, macos, web, windows
borderWidth: Platform.OS === 'ios' ? 0 : 2;
// sử dụng select
borderWidth: Platform.select({ ios: 0, android: 2 });
```

- Điều chỉnh file thành đuôi ios hoặc android để tuỳ chọn component áp dùng tuỳ nền tảng

```js
//file : Title.ios.js và Title.android.js
import Title from '../components/Title';
```

# React Navigation

### Navigation?

- Không sử dụng url, điều hướng bằng cách chạm vào button back, hoặc các button chuyển hướng sang màn hình khác.

### Cài đặt react-navigation

- `npm install @react-navigation/native`
- Cài thêm 1 số gói theo docs: https://reactnavigation.org/docs/getting-started/

- import `NavigationContainer` và dùng component bao quanh toàn bộ component

```js
import { NavigationContainer } from '@react-navigation/native';
```

### Cài đặt native-stack: `npm install @react-navigation/stack`

#### Component: Stack

```js
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();
```

- Sử dụng `Stack.Navigator` để quản lý các screen và `Stack.Screen` để đăng ký 1 screen

```js
<NavigationContainer>
  <Stack.Navigator>
    <Stack.Screen name="MealsCategories" component={CategoriesScreen} />
    <Stack.Screen name="MealsOverview" component={MealsOverviewScreen} />
    <Stack.Screen name="MealDetail" component={MealDetailScreen} />
  </Stack.Navigator>
</NavigationContainer>
```

- props của `Stack.Screen`:

  - `name`: đặt tên có screen như là id cho screen
  - `component`: component screen

- Mặc đinh `Stack.Navigator` cung cấp 1 header mặc định chứa 1 title (là name của screen đang mở) và button back (khi chuyển hướng sang screen khác giúp quay lại screen ở stack trước đó)
- `Stack.Screen` đầu tiên trong `Stack.Navigator` là màn hình hiển thị khi khởi động app. `Stack.Navigator` cung cấp prop `initialRouteName` nhận giá trị là name của screen để làm màn hình hiển thị lúc khởi động

- `Stack.Screen` cung cấp cho các component screen prop `navigation` sử dụng để điều hướng giữa các screen.

```js
// cung cấp name screen cho method navigate để chuyển hướng
// chuyển hướng sang screen MealsOverview
navigation.navigate('MealsOverview');
```

- prop `navigation chỉ được truyền cho các component được gói trong `Stack.Screen`. Để điều hướng trong các component con, các component nested sử dụng `useNavigation()`

```js
import { useNavigation } from '@react-navigation/native';

// sử dụng navigation để điều hướng
const navigation = useNavigation();
```

#### Component Native Stack

- Native Stack sử dụng các native-platform-element, cung cấp các tích hợp gần hơn với các thành phần native của từng nền tảng, giúp tối ưu hóa hiệu suất và trải nghiệm người dùng.
- Có các tính năng như thêm tiêu đề động, hiệu ứng chuyển đổi mượt mà, và khả năng sử dụng tính năng native nâng cao như react-native-screens.

### Truyền data qua các screen

- prop `navigation` sử dụng method `navigate()` sẽ nhận vào tham số thứ 2 là 1 object, từ đó truyền được giá trị qua screen khác

```js
navigation.navigate('MealsOverview', { mealId: 2 });
```

- để trích xuất giá trị truyền vào navigation, sử dụng prop `route` của mỗi screen

```js
const mealId = route.params.mealId;
```

- sử dụng hook `useRoute` cũng lấy được các giá trị, và có thể lấy được trong các component nested

```js
import { useRoute } from '@react-navigation/native';

const route = useRoute();
```

### Style cho header mặc định được cung cấp bởi Stack

- prop `options` của `Stack.Screen` nhận 1 object giúp thay đổi nhiều options của screen: `https://reactnavigation.org/docs/stack-navigator#options`, trong đó có `header`: `https://reactnavigation.org/docs/stack-navigator#header-related-options`

```js
<Stack.Screen
  name="MealsCategories"
  component={CategoriesScreen}
  options={{
    // đặt tiêu đề cho screen
    title: 'All Categories',
    // style cho header
    headerStyle: { backgroundColor: '#555' },
    // style cho icon back, cho title
    headerTintColor: 'white',
    // style cho content của screen
    contentStyle: { backgroundColor: '#aaa' },
    //headerShown: false,
  }}
/>
```

- cách style trên chỉ cho từng screen, muốn áp dụng 1 style cho tất cả screen, sử dụng prop `screenOptions` cho `Stack.Navigator`, tuy nhiên style riên cho từng screen vẫn được ưu tiên

```js
<Stack.Navigator
  screenOptions={{
    headerStyle: { backgroundColor: '#351401' },
    headerTintColor: 'white',
    contentStyle: { backgroundColor: '#3f2f25' },
  }}
>
  <Stack.Screen
    name="MealDetail"
    component={MealDetailScreen}
    options={{
      title: 'About the Meal',
    }}
  />
</Stack.Navigator>
```

- prop `options` của `Stack.Screen` cũng nhận 1 function tham số là object 2 thuộc tính: route, navigation. function thực thi bất cứ khi nào screen active, return về object chứa các options

- `options` cho phép thuộc tính:
  - `presentation` : card | modal | containedModal | formSheet ... cho phép kiểm soát cách load screen, có thể screen như 1 card, như 1 modal
  - `title` : thay đổi title
  - `headerRight` : nhận vào function trả về JSX đặt ở vị trí bên phải header
    ....

```js
<Stack.Screen
  name="MealsOverviewScreen"
  component={MealsOverviewScreen}
  options={({ route, navigation }) => {
    categoryId = route.params.categoryId;
    return { title: categoryId };
  }}
/>
```

- tuỳ chỉnh options cũng có thể ở chính component screen

```js
function MealsOverviewScreen = ({route, navigation}) => {
  const categoryId = route.params.categoryId;

  // setOptions() là 1 side effect phụ thuộc vào route và navigation nên cần bọc trong useEffect
  useLayoutEffect(() => {
    const categoryTitle = CATEGORIES.find((category) => category.id === categoryId).title;
    navigation.setOptions({title: categoryTitle});
    }, [categoryId, navigation])

}
```

- thêm component vào header

```js
<Stack.Screen
  name="MealDetail"
  component={MealDetailScreen}
  options={{
    // thêm component vào bên phải header
    // cũng có thể tuỳ chỉnh trong screen component thông qua prop navigation.setOptions()
    headerRight: () => {
      return <Button title="Tap me!" />;
    },
  }}
/>
```

### Drawer

- thêm 1 ngăn kéo ở bên side của app

```js
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

export default function App() {
  return (
    <Drawer.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: '#351401' },
        headerTintColor: 'white',
        sceneContainerStyle: { backgroundColor: '#3f2f25' },
        drawerContentStyle: { backgroundColor: '#351401' },
        drawerInactiveTintColor: 'white',
        drawerActiveTintColor: '#351401',
        drawerActiveBackgroundColor: '#e4baa1',
      }}
    >
      <Drawer.Screen
        name="Categories"
        component={CategoriesScreen}
        options={{
          title: 'All Categories',
          drawerIcon: ({ color, size }) => <Ionicons name="list" color={color} size={size} />,
        }}
      />
      <Drawer.Screen
        name="Favorites"
        component={FavoritesScreen}
        options={{
          // thuộc tính color nhận giá trị từ drawerTintColor
          drawerIcon: ({ color, size }) => <Ionicons name="star" color={color} size={size} />,
        }}
      />
    </Drawer.Navigator>
  );
}
```

- Mặc đinh `Drawer` cung cấp 1 button để toggle việc đóng mở drawer, tuy nhiên có thể sử dụng prop `navigation` ở mỗi component đăng ký làm Screen gọi đến function ``.toggleDrawer()` để mở drawer

```js
navigation.toggleDrawer();
```

### Bottom Tabs

- Hiển thi các tab ở dưới screen

```js
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function MyTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}
```

### Nesting Navigators

- kết hợp các `Navigators` với nhau: thiết lập 1 `Navigator` trở thành 1 `Screen` của 1 `Navigator` khác
- kết hợp này có thể khiến các giao diện mặc định của các `Navigator` xuất hiện cùng lúc. Ex: xuất hiện 2 header nếu kết hợp `Drawer` và `Stack`

```js
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createDrawerNavigator } from '@react-navigation/drawer';

const Stack = createNativeStackNavigator();
const Drawer = createDrawerNavigator();

function DrawerNavigator() {
  return (
    <Drawer.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: '#351401' },
        headerTintColor: 'white',
        sceneContainerStyle: { backgroundColor: '#3f2f25' },
        drawerContentStyle: { backgroundColor: '#351401' },
        drawerInactiveTintColor: 'white',
        drawerActiveTintColor: '#351401',
        drawerActiveBackgroundColor: '#e4baa1',
      }}
    >
      <Drawer.Screen
        name="Categories"
        component={CategoriesScreen}
        options={{
          title: 'All Categories',
          drawerIcon: ({ color, size }) => <Ionicons name="list" color={color} size={size} />,
        }}
      />
      <Drawer.Screen
        name="Favorites"
        component={FavoritesScreen}
        options={{
          drawerIcon: ({ color, size }) => <Ionicons name="star" color={color} size={size} />,
        }}
      />
    </Drawer.Navigator>
  );
}

export default function App() {
  return (
    <>
      <StatusBar style="light" />
      <NavigationContainer>
        <Stack.Navigator
          screenOptions={{
            headerStyle: { backgroundColor: '#351401' },
            headerTintColor: 'white',
            contentStyle: { backgroundColor: '#3f2f25' },
          }}
        >
          <Stack.Screen
            name="Drawer"
            component={DrawerNavigator}
            options={{
              // ẩn header để không xuất hiện 2 header cùng lúc của cả Drawer và Stack
              headerShown: false,
            }}
          />
          <Stack.Screen name="MealsOverview" component={MealsOverviewScreen} />
          <Stack.Screen
            name="MealDetail"
            component={MealDetailScreen}
            options={{
              title: 'About the Meal',
            }}
          />
        </Stack.Navigator>
      </NavigationContainer>
    </>
  );
}
```

# Create context dùng chung

```js
import { createContext, useState } from 'react';

// Tạo component context và định nghĩa các state
export const FavoritesContext = createContext({
  ids: [],
  addFavorite: (id) => {},
  removeFavorite: (id) => {},
});

// Tạo component Provider,nơi chứa các state, các function thay đổi state
function FavoritesContextProvider({ children }) {
  const [favoriteMealIds, setFavoriteMealIds] = useState([]);

  function addFavorite(id) {
    setFavoriteMealIds((currentFavIds) => [...currentFavIds, id]);
  }

  function removeFavorite(id) {
    setFavoriteMealIds((currentFavIds) => currentFavIds.filter((mealId) => mealId !== id));
  }

  const value = {
    ids: favoriteMealIds,
    addFavorite: addFavorite,
    removeFavorite: removeFavorite,
  };

  // cung cấp giá trị khởi tạo ban đầu cho các state dùng chung
  return <FavoritesContext.Provider value={value}>{children}</FavoritesContext.Provider>;
}

export default FavoritesContextProvider;

/*
 * Sử dụng component Provider bao bọc lấy component cần sử dụng chung state
 * Truy xuất value của Provider bằng cách gọi đến `const Ctx = useContext(FavoritesContext)`
 */
```

# Loading Spinner

- `ActivityIndicator` là component loading được cung cấp sẵn bởi react-native

# User Authentication

# Camera Picker

### expo-camera

- `expo-camera` cho phép custom giao diện chụp ảnh và nhiều thứ khác từ chọn ảnh, chụp ảnh, điều khiển tự động lấy nét, thu phóng,.... Giúp xây dựng giao diện người dùng dựa trên camera

### expo-image-picker

- `expo-image-picker` phù hợp với 1 số tác vụ đơn giản, chỉ chọn ảnh, hoặc chụp ảnh theo cách mặc định

```cmd
npx expo install expo-image-picker
```

- trên android tự động yêu cầu cấu hình quyền, nhưng ios không yêu cầu, cần tự quản lý

- cần được cấu hình trong file `app.js`, những thuộc tính để cấu hình:
  - `photosPermission`
  - `cameraPermission`
  - `microphonePermission`

```js
// thêm vào file app.json
{
  "expo": {
    "plugins": [
      [
        "expo-image-picker",
        {
          // các thuộc tính cần cấu hình : đưa ra message để yêu cầu quyền sử dụng native device với người dùng
          "photosPermission": "The app accesses your photos to let you share them with your friends."
        }
      ]
    ]
  }
}
```

```js
import { Alert, Button, View } from 'react-native';
import { launchCameraAsync, useCameraPermissions, PermissionStatus } from 'expo-image-picker';

function ImagePicker() {
  const [pickedImage, setPickedImage] = useState();
  const [cameraPermissionInformation, requestPermission] = useCameraPermissions();
  // useCameraPermissions dùng để yêu cầu quyền từ ios

  async function verifyPermissions() {
    // kiểm tra quyền đã cấp, nếu chưa xác định thì cần đợi kiểm tra bằng requestPermission và trả lọi permission
    if (cameraPermissionInformation.status === PermissionStatus.UNDETERMINED) {
      const permissionResponse = await requestPermission();

      return permissionResponse.granted;
    }

    if (cameraPermissionInformation.status === PermissionStatus.DENIED) {
      Alert.alert('Insufficient Permissions!', 'You need to grant camera permissions to use this app.');
      return false;
    }

    return true;
  }

  async function takeImageHandler() {
    const hasPermission = await verifyPermissions();

    if (!hasPermission) {
      return;
    }

    // sử dụng launchCameraAsync cùng với 1 số options có thể cầu hình: https://docs.expo.dev/versions/latest/sdk/imagepicker/#imagepickeroptions
    const image = await launchCameraAsync({
      // cho phép chỉnh sửa ảnh chức khi xác nhận
      allowsEditing: true,
      // tỷ lệ ảnh nhận được
      aspect: [16, 9],
      // hạn chế chất lượng nhận vào của ảnh so với ảnh chụp được
      quality: 0.5,
    });
    setPickedImage(image.uri);
  }

  let imagePreview = <Text>No image taken yet.</Text>;

  if (pickedImage) {
    imagePreview = <Image style={styles.image} source={{ uri: pickedImage }} />;
  }

  return (
    <View>
      <View style={styles.imagePreview}>{imagePreview}</View>
      <Button title="Take Image" onPress={takeImageHandler} />
    </View>
  );
}

export default ImagePicker;
```

# Location Picker

- có thể cung cấp location theo 2 cách chọn:
  - định vị người dùng qua gps
  - cho phép chọn vị trí trên map

### `expo-location`

```cmd
npm install expo-location
```

- location yêu cầu 1 số quyền, trong đó có việc lấy vị trí người dùng khi ứng dụng ở chế độ nền đặc biệt là với IOS

- tuy nhiên ở cả 2 dòng thiết bị khi truy cập đến location đều cần yêu cầu quyền

- fetch location cho người dùng android khá rắc rối, có thể bị crash.

```js
import { useEffect, useState } from 'react';
import { Alert, View, StyleSheet, Image, Text } from 'react-native';
import { getCurrentPositionAsync, useForegroundPermissions, PermissionStatus } from 'expo-location';
import { useNavigation, useRoute, useIsFocused } from '@react-navigation/native';

import { Colors } from '../../constants/colors';
import OutlinedButton from '../UI/OutlinedButton';
import { getAddress, getMapPreview } from '../../util/location';

function LocationPicker({ onPickLocation }) {
  // tạo state lưu vị trí được chọn
  const [pickedLocation, setPickedLocation] = useState();
  // useIsFocused là 1 hook được cung cấp bởi react-native, mang giá trị là true nếu màn hình đang được active
  const isFocused = useIsFocused();

  const navigation = useNavigation();
  const route = useRoute();

  // lấy ra quyền truy cập location
  const [locationPermissionInformation, requestPermission] = useForegroundPermissions();

  // check param để set state vị trí, param nhận được ở screen Map, khi đã chọn vị trí trên map xong chuyển hướng về
  // tuy nhiên khi go back từ screen Map về lại screen AddPlace, thì các component của AddPlace không được re-render lại
  // vấn đề này do sử dụng Stack, 1 màn hình mới được bật lên chỉ đơn giản là đưa lên đầu Stack, sau khi go back thì màn hình này được loại bỏ nhưng màn hình trước nó vẫn giữu nguyên
  // sử dụng useIsFocused để biết màn hình đã từng chuyển màn khác chưa, dựa vào đó re-render
  useEffect(() => {
    if (isFocused && route.params) {
      const mapPickedLocation = {
        lat: route.params.pickedLat,
        lng: route.params.pickedLng,
      };
      setPickedLocation(mapPickedLocation);
    }
  }, [route, isFocused]);

  // đưa thông tin vị trí chọn được lên component cha
  useEffect(() => {
    async function handleLocation() {
      if (pickedLocation) {
        const address = await getAddress(pickedLocation.lat, pickedLocation.lng);
        onPickLocation({ ...pickedLocation, address: address });
      }
    }

    handleLocation();
  }, [pickedLocation, onPickLocation]);

  // xác định quyền truy cập location
  async function verifyPermissions() {
    if (locationPermissionInformation.status === PermissionStatus.UNDETERMINED) {
      const permissionResponse = await requestPermission();

      return permissionResponse.granted;
    }

    if (locationPermissionInformation.status === PermissionStatus.DENIED) {
      Alert.alert('Insufficient Permissions!', 'You need to grant location permissions to use this app.');
      return false;
    }

    return true;
  }

  // hàm lấy ra location hiện tại, đưa vào state vị trí để hiển thị map
  async function getLocationHandler() {
    const hasPermission = await verifyPermissions();

    if (!hasPermission) {
      return;
    }

    // lấy ra vị trí hiện tại của user
    const location = await getCurrentPositionAsync();
    // getCurrentPositionAsync() cũng nhận vào 1 số options về độ chính xác, thời gian mỗi lền fetch vị trí lại, khoảng cách
    setPickedLocation({
      lat: location.coords.latitude,
      lng: location.coords.longitude,
    });
  }

  function pickOnMapHandler() {
    navigation.navigate('Map');
  }

  let locationPreview = <Text>No location picked yet.</Text>;

  // nếu có state vị trí thì hiển thị map được cung cấp bởi google api qua lat và lng
  if (pickedLocation) {
    locationPreview = (
      <Image
        style={styles.image}
        source={{
          uri: getMapPreview(pickedLocation.lat, pickedLocation.lng),
        }}
      />
    );
  }

  return (
    <View>
      <View style={styles.mapPreview}>{locationPreview}</View>
      <View style={styles.actions}>
        <OutlinedButton icon="location" onPress={getLocationHandler}>
          Locate User
        </OutlinedButton>
        <OutlinedButton icon="map" onPress={pickOnMapHandler}>
          Pick on Map
        </OutlinedButton>
      </View>
    </View>
  );
}

//////////////////////////////////////////////////////////////
// file sử dụng google api lấy vị trí và lấy hình ảnh bản đồ vị trí dựa trên lng và lat
const GOOGLE_API_KEY = 'AIzaSyD2mP_sDlloOcwv6OLmYtATOJIcnDhcZcg';

export function getMapPreview(lat, lng) {
  // map static api
  const imagePreviewUrl = `https://maps.googleapis.com/maps/api/staticmap?center=${lat},${lng}&zoom=14&size=400x200&maptype=roadmap&markers=color:red%7Clabel:S%7C${lat},${lng}&key=${GOOGLE_API_KEY}`;
  return imagePreviewUrl;
}

// geocoding api
export async function getAddress(lat, lng) {
  const url = `https://maps.googleapis.com/maps/api/geocode/json?latlng=${lat},${lng}&key=${GOOGLE_API_KEY}`;
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error('Failed to fetch address!');
  }

  const data = await response.json();
  const address = data.results[0].formatted_address;
  return address;
}
```

- 1 api cụ thể do google map cung cấp giúp hiển thị map theo lat và lng, cần goole account và credit card: https://console.cloud.google.com/google/maps-apis/discover?project=eng-plasma-366811

- thư viện `react-native-maps` giúp hiển thị map và chọn vị trí
  https://docs.expo.dev/versions/latest/sdk/map-view/

```js
import { useCallback, useLayoutEffect, useState } from 'react';
import { Alert, StyleSheet } from 'react-native';
import MapView, { Marker } from 'react-native-maps';

import IconButton from '../components/UI/IconButton';

function Map({ navigation, route }) {
  // có thể nhận được vị trí từ  AddPlace screen
  const initialLocation = route.params && {
    lat: route.params.initialLat,
    lng: route.params.initialLng,
  };

  const [selectedLocation, setSelectedLocation] = useState(initialLocation);

  // object về initialRegion được truyền vào component Map
  const region = {
    latitude: initialLocation ? initialLocation.lat : 37.78,
    longitude: initialLocation ? initialLocation.lng : -122.43,
    latitudeDelta: 0.0922,
    longitudeDelta: 0.0421,
  };

  // function lấy ra lat và lng từ sự kiện chọn vị trí trên map
  function selectLocationHandler(event) {
    if (initialLocation) {
      return;
    }
    const lat = event.nativeEvent.coordinate.latitude;
    const lng = event.nativeEvent.coordinate.longitude;

    setSelectedLocation({ lat: lat, lng: lng });
  }

  // function giúp navigate sang screen addPlace
  const savePickedLocationHandler = useCallback(() => {
    if (!selectedLocation) {
      Alert.alert('No location picked!', 'You have to pick a location (by tapping on the map) first!');
      return;
    }

    navigation.navigate('AddPlace', {
      pickedLat: selectedLocation.lat,
      pickedLng: selectedLocation.lng,
    });
  }, [navigation, selectedLocation]);

  // thêm 1 button để chuyển hướng từ screen map, chọn vị trí về lại màn hình addPlace
  // do sử dụng savePickedLocationHandler là 1 function làm dependency nên có khả năng mỗi lần re-render tạo lại function và tiếp tụ re-render vô hạn => sử dụng useCallback bao bọc lấy function
  useLayoutEffect(() => {
    if (initialLocation) {
      return;
    }
    navigation.setOptions({
      headerRight: ({ tintColor }) => (
        <IconButton icon="save" size={24} color={tintColor} onPress={savePickedLocationHandler} />
      ),
    });
  }, [navigation, savePickedLocationHandler, initialLocation]);

  return (
    <MapView style={styles.map} initialRegion={region} onPress={selectLocationHandler}>
      {selectedLocation && (
        <Marker
          title="Picked Location"
          coordinate={{
            latitude: selectedLocation.lat,
            longitude: selectedLocation.lng,
          }}
        />
      )}
    </MapView>
  );
}

export default Map;

const styles = StyleSheet.create({
  map: {
    flex: 1,
  },
});
```

# Database lưu trên local của thiết bị: SQLite

```npm
expo install expo-sqlite
```

- Db được tạo ra tồn tại đến khi gỡ cài đặt app

```js
import * as SQLite from 'expo-sqlite';

import { Place } from '../models/place';

// tạo database với tên places.db
const database = SQLite.openDatabase('places.db');

// function init() được thực thi trong useEffect khi lần đầu khởi động app
// init() trả về promise => tạo 1 state để check trạng thái tạo db, useEffect(() => init.then(setState(true)), [])
// dựa vào state, = false hiện loading, = true render component App
export function init() {
  const promise = new Promise((resolve, reject) => {
    // transaction thực hiện 1 query trả về 1 promise
    // tham số nhận vào 1 function (transactionObj) => {}
    database.transaction((tx) => {
      // executeSql: hàm thực thi query
      // (sqlStatmenet, arg[string, number], callback, errorCallback)
      tx.executeSql(
        `CREATE TABLE IF NOT EXISTS places (
          id INTEGER PRIMARY KEY NOT NULL,
          title TEXT NOT NULL,
          imageUri TEXT NOT NULL,
          address TEXT NOT NULL,
          lat REAL NOT NULL,
          lng REAL NOT NULL
        )`,
        [],
        () => {
          resolve();
        },
        (_, error) => {
          reject(error);
        }
      );
    });
  });

  return promise;
}

export function insertPlace(place) {
  const promise = new Promise((resolve, reject) => {
    database.transaction((tx) => {
      tx.executeSql(
        `INSERT INTO places (title, imageUri, address, lat, lng) VALUES (?, ?, ?, ?, ?)`,
        [place.title, place.imageUri, place.address, place.location.lat, place.location.lng],
        (_, result) => {
          resolve(result);
        },
        (_, error) => {
          reject(error);
        }
      );
    });
  });

  return promise;
}

export function fetchPlaces() {
  const promise = new Promise((resolve, reject) => {
    database.transaction((tx) => {
      tx.executeSql(
        'SELECT * FROM places',
        [],
        (_, result) => {
          const places = [];

          for (const dp of result.rows._array) {
            places.push(
              new Place(
                dp.title,
                dp.imageUri,
                {
                  address: dp.address,
                  lat: dp.lat,
                  lng: dp.lng,
                },
                dp.id
              )
            );
          }
          resolve(places);
        },
        (_, error) => {
          reject(error);
        }
      );
    });
  });

  return promise;
}

export function fetchPlaceDetails(id) {
  const promise = new Promise((resolve, reject) => {
    database.transaction((tx) => {
      tx.executeSql(
        'SELECT * FROM places WHERE id = ?',
        [id],
        (_, result) => {
          const dbPlace = result.rows._array[0];
          const place = new Place(
            dbPlace.title,
            dbPlace.imageUri,
            { lat: dbPlace.lat, lng: dbPlace.lng, address: dbPlace.address },
            dbPlace.id
          );
          resolve(place);
        },
        (_, error) => {
          reject(error);
        }
      );
    });
  });

  return promise;
}
```

# Notification (Expo)

- `expo install expo-notification`
- https://docs.expo.dev/versions/latest/sdk/notifications/
- thêm cấu hình sau vào app.json

```js
{
  "expo": {
    "plugins": [
      [
        "expo-notifications",
        {
          // cấu hình về icon, color, sound của notification
          // icon và color chỉ cho android
          //
          "icon": "./local/assets/notification-icon.png",
          "color": "#ffffff",
          "sounds": [
            "./local/assets/notification-sound.wav",
            "./local/assets/notification-sound-other.wav"
          ]
        }
      ]
    ]
  }
}
```

-

```js
import { useEffect } from 'react';
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Button, View, Alert, Platform } from 'react-native';
import * as Notifications from 'expo-notifications';

// thiết lập 1 số logic để xử lý thông báo
// cần thiết để ứng dụng biết xử lý các thông báo và hiển thị thông báo
Notifications.setNotificationHandler({
  // trigger mỗi khi nhận được thông báo
  handleNotification: async () => {
    return {
      // kiểm soát việc phát âm thanh khi thông báo
      shouldPlaySound: false,
      // việc thêm icon thông báo vào icon của app
      shouldSetBadge: false,
      // cảnh báo khi nhận được thông báo
      shouldShowAlert: true,
    };
  },
  // handleError, handleSuccess: tigger khi xử lý thông báo thành công hay thất bại
});

export default function App() {
  useEffect(() => {
    // nếu muốn xuất bản app độc lập khởi app Expo go, app cần xin cấp quyền để khai thác các tính năng liên quan đến thông báo
    async function configurePushNotifications() {
      const { status } = await Notifications.getPermissionsAsync();
      let finalStatus = status;

      // nếu không có quyền => yêu cầu quyền
      if (finalStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      // nếu vẫn không cấp quyền thì hiển thị cảnh báo
      if (finalStatus !== 'granted') {
        Alert.alert('Permission required', 'Push notifications need the appropriate permissions.');
        // không tiếp tục thực thi code đằng sau
        return;
      }

      // lấy token của thiết bị ngay khi lần đầu khởi động app
      // token sẽ được lấy và lưu trong database
      const pushTokenData = await Notifications.getExpoPushTokenAsync();
      console.log(pushTokenData);

      // kiểm tra thietês bịj cós là android
      if (Platform.OS === 'android') {
        // cần thiết để android device xác nhận channel nào được nhận thông báo
        Notifications.setNotificationChannelAsync('default', {
          name: 'default',
          // mức độ quan trọng của thông báo
          importance: Notifications.AndroidImportance.DEFAULT,
        });
      }
    }

    configurePushNotifications();
  }, []);

  useEffect(() => {
    // cho phép đăng ký 1 chức năng xử lý sự kiện, sẽ được thực thi khi thiết bị nhận thông báo mới
    const subscription1 = Notifications.addNotificationReceivedListener((notification) => {
      console.log('NOTIFICATION RECEIVED');
      console.log(notification);
      const userName = notification.request.content.data.userName;
      console.log(userName);
    });

    // đăng ký sự kiện sẽ thực thi khi user phản hồi lại thông báo
    const subscription2 = Notifications.addNotificationResponseReceivedListener((response) => {
      console.log('NOTIFICATION RESPONSE RECEIVED');
      console.log(response);
      const userName = response.notification.request.content.data.userName;
      console.log(userName);
    });

    return () => {
      // xoá sự kiện được đăng ký mỗi khi component không sử dụng nữa
      subscription1.remove();
      subscription2.remove();
    };
  }, []);

  // lên lịch cho notification
  function scheduleNotificationHandler() {
    // trả về 1 promise
    Notifications.scheduleNotificationAsync({
      // định cấu hình nội dung cho thông báo
      // các thuộc tính có thể đặt cho content: https://docs.expo.dev/versions/latest/sdk/notifications/#notificationcontentinput
      content: {
        title: 'My first local notification',
        body: 'This is the body of the notification.',
        // data không hiển thị khi thông báo mở, nhưng có thể trích xuất khi xử lý thông báo
        data: { userName: 'Max' },
      },
      // thời gian khi nào thông báo được giử
      // thuộc tính có thể cấu hình cho trigger: https://docs.expo.dev/versions/latest/sdk/notifications/#notificationtriggerinput
      trigger: {
        seconds: 5,
      },
      // indentifier
    });
  }

  function sendPushNotificationHandler() {
    fetch('https://exp.host/--/api/v2/push/send', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        to: '[<Your Device Push Token>]',
        // token sẽ được lưu trong database
        title: 'Test - sent from a device!',
        body: 'This is a test!',
      }),
    });
  }

  return (
    <View style={styles.container}>
      <Button title="Schedule Notification" onPress={scheduleNotificationHandler} />
      <Button title="Send Push Notification" onPress={sendPushNotificationHandler} />
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

- Push Notification: tương tác với ứng dụng trên thiết bị khác cùng cài đặt app, như thông báo giử tin nhắn đến thiết bị khác. Thống báo có thể được đẩy tử back end hoặc từ 1 thiết bị tới các thiết bị khác
- không thể trực tiếp giử thông báo do bảo mật và ngăn chặn spam => google và apple buộc sử dụng backend được cung cấp bởi họ để đẩy thông báo
- cần giử 1 http request tới máy chủ của gg hay apple để giử thôong báo đến các thiết bị
- cần thiết bị thực để test chức năng
- `push token` là 1 chuỗi duy nhất với mỗi thiết bị, đóng vai trò là địa chỉ để giửu thông báo đến
