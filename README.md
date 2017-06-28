# AutoRxPreferences
Annotation processing for RxPreferences

## Introduction
RxPreferences by f2prateek is great library but it has a few downsides 
1. You have to mess around with the pref keys instead of access your preferences by name which feels more natural
2. If you want to use your own default values you have to add them to every method you write
3. Saving custom objects is to complicated

AutoRxPreferences solves those issues by using annotation processing.

## Download
```groovy
// in your root gradle
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```

```groovy
// in your module
dependencies {
	 compile 'com.github.IVIanuu.AutoRxPreferences:autorxpreferences:LATEST-VERSION'
         annotationProcessor 'com.github.IVIanuu.AutoRxPreferences:autorxpreferences-processor:LATEST-VERSION'
}
```

## Usage

1. Create a class and annotate it with the @Preferences annotation.
2. Create some fields and annotate them with the @Key annotation.

The class should be look like this

```java
@Preferences
class MyPreferences {
    @Key String accessToken;
    @Key UserData userData;
    @Key Boolean isLoggedIn = false; // this will be default value
}
```

This is the class which will be generated by AutoRxPreferences
```java
public final class MyPreferences_ extends MyPreferences {
  private final RxSharedPreferences rxSharedPreferences;

  private final Preference.Converter<UserData> userDataConverter;

  private MyPreferences_(Context context, Gson gson) {
    SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
    this.rxSharedPreferences = RxSharedPreferences.create(sharedPreferences);
    this.userDataConverter = new UserDataConverter(gson);
  }

  public static MyPreferences_ create(@NonNull Context context) {
    return create(context, new Gson());
  }

  public static MyPreferences_ create(@NonNull Context context, @NonNull Gson gson) {
    return new MyPreferences_(context, gson);
  }

  @NonNull
  public RxSharedPreferences getRxSharedPreferences() {
    return rxSharedPreferences;
  }

  @NonNull
  public Preference<String> accessToken() {
    if (accessToken != null) {
      return accessToken(accessToken);
    } else {
      return rxSharedPreferences.getString("accessToken");
    }
  }

  @NonNull
  public Preference<String> accessToken(@NonNull String defaultValue) {
    return rxSharedPreferences.getString("access_token", defaultValue);
  }

  @NonNull
  public Preference<UserData> userData() {
    if(userData == null) {
      throw new IllegalStateException("userData has no default value");
    }
    return userData(userData);
  }

  @NonNull
  public Preference<UserData> userData(@NonNull UserData defaultValue) {
    return userData(defaultValue, userDataConverter);
  }

  @NonNull
  public Preference<UserData> userData(@NonNull UserData defaultValue,
      @NonNull Preference.Converter<UserData> converter) {
    return rxSharedPreferences.getObject("user_data", defaultValue, converter);
  }

  @NonNull
  public Preference<Boolean> isLoggedIn() {
    if (isLoggedIn != null) {
      return isLoggedIn(isLoggedIn);
    } else {
      return rxSharedPreferences.getBoolean("isLoggedIn");
    }
  }

  @NonNull
  public Preference<Boolean> isLoggedIn(@NonNull Boolean defaultValue) {
    return rxSharedPreferences.getBoolean("is_logged_in", defaultValue);
  }

  private static final class UserDataConverter implements Preference.Converter<UserData> {
    private final Gson gson;

    private UserDataConverter(Gson gson) {
      this.gson = gson;
    }

    @NonNull
    @Override
    public UserData deserialize(String serialized) {
      return gson.fromJson(serialized, UserData.class);
    }

    @NonNull
    @Override
    public String serialize(UserData value) {
      return gson.toJson(value);
    }
  }
}
```

This saves you tones of boilerplate code. Let's see what you got for free here
1. You can access your preferences by name not by key 
2. Default values will be applied to every call if they are not null (You can do it anyway if you want)
3. A converter powered by gson is automatically created for you (You can pass your own Gson instance in the create method)

## License

```
Copyright 2017 Manuel Wrage

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
 
http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
