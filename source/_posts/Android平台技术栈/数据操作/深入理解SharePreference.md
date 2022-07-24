---
title: 深入理解 SharePreference 
date: 2022-04-10 13:38:32
tags: SharePreference
categories: Android平台技术栈
---

SharedPreferences(简称SP)是Android中很常用的数据存储方式，SP采用key-value（键值对）形式, 主要用于轻量级的数据存储。SharedPreferences 对象指向包含键值对的文件，并提供读写这些键值对的简单方法。

> > SharedPreferences API 用于读写键值对，不要将它们与 Preference API 混淆，后者可帮助您构建用于显示应用设置的界面（虽然它们也使用 SharedPreferences 保存用户设置）

## 一、概述

SharedPreferences 是 Android 平台为应用开发者提供的一个轻量级的存储辅助类，用来保存应用的一些常用配置，它提供了 putString()、putString(Set\<String\>)、putInt()、putLong()、putFloat()、putBoolean() 六种数据类型。数据最终是以 XML 形式进行存储。在应用中通常做一些简单数据的持久化存储。SharedPreferences 作为一个轻量级存储，所以就限制了它的使用场景，如果对它使用不当可能会引发“严重后果”。

SP采用xml文件格式来保存数据, 该文件所在目录位于 `/data/data/${pkgName}/shared_prefs/` 。

## 二、SharePreferences接口

SharePreferences 是一个接口，其内部包含2个接口，一个是提供编辑能力的Editor接口，另一个是提供SharePreference变更通知的监听器接口。SharePreferences接口提供了一系列从sp xml文件中读取键值的方法以及注册sp变更的监听器，而对sp xml文件的写入(编辑) 能力是通过其内部接口Editor来提供的。

> > 设计思考：为什么不将Editor提供的put系列接口直接放在SharePreferences接口中，而是要将写入的接口放入Editor？

### 2.1. SharePreference接口方法

```js
// 读取key对应的值
Map<String, ?> getAll();
String getString(String key, @Nullable String defValue);
Set<String> getStringSet(String key, @Nullable Set<String> defValues);
int getInt(String key, int defValue);
long getLong(String key, long defValue);
float getFloat(String key, float defValue);
boolean getBoolean(String key, boolean defValue);

// 判断key是否存在
boolean contains(String key);
// 获取SP编辑器，来获取写入SP能力
Editor edit();
// 注册和注销，当该SP中的key-value变更时将会回调OnSharedPreferenceChangeListener的方法
void registerOnSharedPreferenceChangeListener(OnSharedPreferenceChangeListener listener);
void unregisterOnSharedPreferenceChangeListener(OnSharedPreferenceChangeListener listener);
```

### 2.2. Editor接口

```js
// 向sp xml文件put 写入key-value键值对
Editor putString(String key, @Nullable String value);
Editor putStringSet(String key, @Nullable Set<String> values);
Editor putInt(String key, int value);
Editor putLong(String key, long value);
Editor putFloat(String key, float value);
Editor putBoolean(String key, boolean value);

// 移除指定key-value
Editor remove(String key);
// 清空sp xml
Editor clear();
// 同步提交，直接写入磁盘文件
boolean commit();
// 异步提交，先写入内存，最后异步写入到磁盘文件；快速频繁调用可能导致OOM
void apply();
```

## 三、基本使用

SharePreference的使用非常简单，这也是它为什么受开发者青睐的原因之一：

1. 创建SharePreference对象
2. 执行读写操作

### 3.1. 创建SharePreferecne对象

有三种方式：

1. 通过PreferenceManager获取：返回的是读写默认sp文件的SharePreference对象。

	```js
	public static SharedPreferences getDefaultSharedPreferences(Context context) {
		return context.getSharedPreferences(getDefaultSharedPreferencesName(context),
	        	getDefaultSharedPreferencesMode());
	}

	public static String getDefaultSharedPreferencesName(Context context) {
		return context.getPackageName() + "_preferences";
	}
	```

2. 通过Activity获取：Activity提供getPreferences方法， 以当前Activity的类名作为SP的文件名. 即xxxActivity.xml。

	```js
	public SharedPreferences getPreferences(@Context.PreferencesMode int mode) {
		return getSharedPreferences(getLocalClassName(), mode);
	}

	public String getLocalClassName() {
		final String pkg = getPackageName();
		final String cls = mComponent.getClassName();
		int packageLen = pkg.length();
		if (!cls.startsWith(pkg) || cls.length() <= packageLen
	        	|| cls.charAt(packageLen) != '.') {
	    	return cls;
		}
		return cls.substring(packageLen+1);
	}
	```

2. 通过Context获取：Context提供了getSharedPreferences方法，其实现是在ContextImpl类，具体实现在源码分析。

	```js
	// name是sp xml文件名称
	// mode 是模式
	public abstract SharedPreferences getSharedPreferences(String name, @PreferencesMode int mode);
	```

	```js
	// mode 取值
	@IntDef(flag = true, prefix = { "MODE_" }, value = {
	    	MODE_PRIVATE,
	    	MODE_WORLD_READABLE,// 弃用
	    	MODE_WORLD_WRITEABLE, // 弃用
	    	MODE_MULTI_PROCESS, // 弃用
	})
	```

> > 自 API 级别 17 起，`MODE_WORLD_READABLE` 和 `MODE_WORLD_WRITEABLE` 模式已被弃用。 从 Android 7.0（API 级别 24）开始，如果您使用这些模式，Android 会抛出 SecurityException。如果您的应用需要与其他应用共享私有文件，可以通过 `FLAG_GRANT_READ_URI_PERMISSION` 使用 `FileProvider`。

### 3.2. 读写

```js
private void write() {
    SharedPreferences sharedPreferences = getSharedPreferences("jacky", MODE_PRIVATE);
	// 多次写入时，不要多次调用edit()方法，因为每次调用edit方法都会新建一个EditorImpl对象
    SharedPreferences.Editor editor = sharedPreferences.edit();
    editor.putString("name", "Jacky");
    editor.putInt("age", 18);
    editor.apply();
}

private void read() {
    SharedPreferences sharedPreferences = getSharedPreferences("jacky", MODE_PRIVATE);
    int age = sharedPreferences.getInt("age", 0);
    String name = sharedPreferences.getString("name", null);
}
```

## 三、源码解析

![](%E6%88%AA%E5%B1%8F2022-04-10%2013.40.21.png)

![](%E6%88%AA%E5%B1%8F2022-04-10%2013.41.00.png)

### 3.1. 获取SharePreference

以ContextImpl的getSharePreference方法为例：

```js
/**
 * 多层级Map，从应用包名映射到SP文件再映射到SharedPreferenceImpl对象。
 * static变量，全局缓存
 */
private static ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>> sSharedPrefsCache;

/**
 * Map，从SP文件名映射到SP文件File对象
 */
private ArrayMap<String, File> mSharedPrefsPaths;
```

ContextImpl记录着SharedPreferences的重要数据：

- sSharedPrefsCache:以包名为key, 二级key是以SP文件, 以SharedPreferencesImpl为value的嵌套map结构。sSharedPrefsCache是静态类成员变量，每个应用进程是保存唯一一份, 且由ContextImpl.class锁保护。
	- mSharedPrefsPaths:记录所有的SP文件, 以文件名为key, 具体文件为value的map结构;
	- mPreferencesDir:是指SP所在目录, 是指`/data/data//shared_prefs/`

```js
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    // 对于Android 4.4 API 19以下版本，如果name传null值，那么构建的xml文件名就是null.xml
    if (mPackageInfo.getApplicationInfo().targetSdkVersion <
            Build.VERSION_CODES.KITKAT) {
        if (name == null) {
            name = "null";
        }
    }

    File file;
    // 注意：这是一个类锁，当多处并发调用getSharedPreferences时，必须先获取锁对象才可执行，保证对mSharedPrefsPaths操作的并发安全
    synchronized (ContextImpl.class) {
        // mSharedPrefsPaths是一个ArrayMap。它记录所有的SP文件，KEY是文件名，Value是文件名对应的文件File对象
        // 如果为空，则新创建一个ArrayMap
        if (mSharedPrefsPaths == null) {
            mSharedPrefsPaths = new ArrayMap<>();
        }
        // 如果不为null，就先根据文件名试着找到File
        file = mSharedPrefsPaths.get(name);
        // 如果找不到该文件，则调用getSharedPreferencesPath方法去创建File对象
        if (file == null) {
            file = getSharedPreferencesPath(name);
            // 创建好新的File之后存入到map中
            mSharedPrefsPaths.put(name, file);
        }
    }
    return getSharedPreferences(file, mode);
}
```

当用Context调用getSharedPreferences方法时，先根据传入的文件名，去mSharedPrefsPaths ArrayMap中查找是否有对应的File对象。如果找不到File对象，说明此SP文件不存在，那么调用getSharedPreferencesPath去 `/data/data/应用包名/shared_prefs` 路径创建一个新的SP文件；并更新到mSharedPrefsPaths；

File 对象的创建：

```js
@Override
public File getSharedPreferencesPath(String name) {
    // 创建sp文件对象
    return makeFilename(getPreferencesDir(), name + ".xml");
}

// 目录创建
@UnsupportedAppUsage
private File getPreferencesDir() {
    synchronized (mSync) {
        // 如果还没有创建shared_prefs目录，那么就先创建此目录，路径是/data/data/应用包名/shared_prefs
        if (mPreferencesDir == null) {
            mPreferencesDir = new File(getDataDir(), "shared_prefs");
        }
        // 进一步确保此目录存在，如果不存在会调用OS的方法来创建对应权限的目录
        return ensurePrivateDirExists(mPreferencesDir);
    }
}

// 文件创建
private File makeFilename(File base, String name) {
    if (name.indexOf(File.separatorChar) < 0) {
        final File res = new File(base, name);
        // We report as filesystem access here to give us the best shot at
        // detecting apps that will pass the path down to native code.
        BlockGuard.getVmPolicy().onPathAccess(res.getPath());
        return res;
    }
	// 文件名不能包含路径分隔符/，否则抛出异常
    throw new IllegalArgumentException(
            "File " + name + " contains a path separator");
}
```

SharePreference实现类对象SharePreferenceImpl创建：

```js
@Override
public SharedPreferences getSharedPreferences(File file, int mode) {
    // SharedPreferences的实现类
    SharedPreferencesImpl sp;
    // 类锁同步
    synchronized (ContextImpl.class) {
        // 尝试从缓存中获取SharePreferenceImpl对象
        final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();
        sp = cache.get(file);
        // 缓存中没有
        if (sp == null) {
            // 检查传入的MODE是否合法
            checkMode(mode);
            // 出现该问题的原因：targetSdk为androidO及以上、访问内部存储空间、当前设备处于锁定状态。
            // 8.0以后 https://blog.csdn.net/kongqwesd12/article/details/84673654
            // https://www.codeleading.com/article/78852205647/
            if (getApplicationInfo().targetSdkVersion >= android.os.Build.VERSION_CODES.O) {
                if (isCredentialProtectedStorage()
                        && !getSystemService(UserManager.class)
                                .isUserUnlockingOrUnlocked(UserHandle.myUserId())) {
                    throw new IllegalStateException("SharedPreferences in credential encrypted "
                            + "storage are not available until after user is unlocked");
                }
            }
            // 创建SharedPreferencesImpl实现类对象; 一个File对应一个SharedPreferencesImpl对象
            sp = new SharedPreferencesImpl(file, mode);
            // 对象放入缓存，返回SharePreference
            cache.put(file, sp);
            return sp;
        }
    }
    // 如果mode包含MODE_MULTI_PROCESS，且在Android 3.0以前，那么会重新load sp数据
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        // If somebody else (some other process) changed the prefs
        // file behind our back, we reload it.  This has been the
        // historical (if undocumented) behavior.
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```

SharedPreferencesImpl 缓存：

```js
private ArrayMap<File, SharedPreferencesImpl> getSharedPreferencesCacheLocked() {
    // sSharedPrefsCache 是用于缓存，ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>> ;
    if (sSharedPrefsCache == null) {
        sSharedPrefsCache = new ArrayMap<>();
    }
    // 获取应用包名
    final String packageName = getPackageName();
    // 一个应用包名，对应一个ArrayMap<File, SharedPreferencesImpl>，它存储的是sp文件对象到SharedPreferencesImpl对象的映射
    ArrayMap<File, SharedPreferencesImpl> packagePrefs = sSharedPrefsCache.get(packageName);
    // 如果缓存中没有此应用对应的ArrayMap<File, SharedPreferencesImpl> ，那么创建一个, 并放入内存
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
		// 这里的存储是根据包名，保存所有SharedPreferencesImpl集合
        sSharedPrefsCache.put(packageName, packagePrefs);
    }

    return packagePrefs;
}
```

Mode检查：

```js
private void checkMode(int mode) {
    // Android N及以后，禁止使用MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE,否则直接抛出异常
	// MODE_MULTI_PROCESS这种多进程的方式也是Google不推荐的方式,目前已经标记弃用，但是未在代码执行层面抛出异常。后续同样会不再支持, 强烈建议App不用使用该方式来实现多个进程实现 同一个SP文件.
	// 当设置MODE_MULTI_PROCESS模式, 则每次getSharedPreferences过程, 会检查SP文件上次修改时间和文件大小, 一旦修改则会重新从磁盘加载文件.
    if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.N) {
        if ((mode & MODE_WORLD_READABLE) != 0) {
            throw new SecurityException("MODE_WORLD_READABLE no longer supported");
        }
        if ((mode & MODE_WORLD_WRITEABLE) != 0) {
            throw new SecurityException("MODE_WORLD_WRITEABLE no longer supported");
        }
    }
}
```

1. ContextImpl中包含sSharedPrefsCache对象，它是一个多层次的Map，从应用包名映射到SP文件File对象，再映射到SharedPreferenceImpl对象。它是对SharedPreferenceImpl对象的缓存，避免重复创建与销毁带来的开销。**应用包名对应多个SP文件，一个SP文件对应一个SharedPreferenceImpl对象。**
2. 在获取SharePreference对象时，是先尝试从缓存中获取。如果缓存中没有找到与sp文件对应的SharedPreferenceImpl对象，那么说明此sp文件还未加载到内存中，因此就需要创建一个新的SharedPreferenceImpl对象；否则，直接返回缓存中的SharedPreferenceImpl对象。
3. ContextImpl对SP做了全局缓存，因此即使你反复调用getSharedPreferences()，并不会创建重复的SharedPreferencesImpl对象。坏消息是这个缓存并没有任何的trim机制，如果你使用的SP足够多而且其中堆放的数据足够多，你还是可能会遇上内存问题

### 3.2. SharePreferenceImpl 创建与初始化

```js
SharedPreferencesImpl(File file, int mode) {
    mFile = file;
    // 创建用于备份的文件；同名的.bak备份文件用于发生异常时, 可通过备份文件来恢复数据.
    mBackupFile = makeBackupFile(file);
    mMode = mode;
    mLoaded = false;
    mMap = null;
    mThrowable = null;
    // 启动子线程，从磁盘中异步加载sp xml文件中的key-value数据到内存map中
    startLoadFromDisk();
}
```

创建备份文件，备份文件名是后缀为.bak的文件：

```js
static File makeBackupFile(File prefsFile) {
    return new File(prefsFile.getPath() + ".bak");
}
```

异步加载磁盘xml数据：

```js
private void startLoadFromDisk() {
    // 锁同步
    synchronized (mLock) {
        mLoaded = false; // mLoaded用于标记SP文件是否已加载到内存
    }
    // 创建子线程从磁盘加载sp xml中的key-value到内存map中
    // 注意: 这是一个IO操作，要解析xml，如果key-value很多，那么必然导致数据加载慢的问题
    new Thread("SharedPreferencesImpl-load") {
        public void run() {
            loadFromDisk();
        }
    }.start();
}
```

```js
private void loadFromDisk() {
    synchronized (mLock) {
        // 如果已经加载过，那么直接忽略
        if (mLoaded) {
            return;
        }
        // 如果存在备份文件，那么用备份文件替代
        if (mBackupFile.exists()) {
            mFile.delete();
            mBackupFile.renameTo(mFile);
        }
    }

	...

    Map<String, Object> map = null;
	...
    try {
        stat = Os.stat(mFile.getPath());
        // 文件可读
        if (mFile.canRead()) {
            BufferedInputStream str = null;
            try {
                str = new BufferedInputStream(
                        new FileInputStream(mFile), 16 * 1024);
                // 利用xml工具类从xml中读取key-value的map
                map = (Map<String, Object>) XmlUtils.readMapXml(str);
            } 
	...

    synchronized (mLock) {
        // 到这表明xml文件数据全部加载到内存，存入map中了
        mLoaded = true;
		...
        } finally {
			// 注意：这里非常重要，当数据加载完毕后，要唤醒get、put等操作，避免它们阻塞。
            mLock.notifyAll();
        }
    }
}
```

SharePreferenceImpl对象创建时，会进行初始化操作，将mLoaded是否加载完成的标记初始化为false；创建一个备份文件，与sp文件同名，后缀名为.bak；最后再创建一个工作线程将sp xml文件进行解析，将xml中的键值对加载到内存的map中存储。多个线程同时操作一个SharePreferenceImpl对象时，会在其mLock对象上进行同步，以保证线程安全。

### 3.3. 查询数据

以getString为例：

```js
public String getString(String key, @Nullable String defValue) {
    // SharePreferencesImpl中get系列方法与注册注销、loadFromDisk等方法都是互斥的，必须要拿到锁后才可进行
    synchronized (mLock) {
        // 重要：getString方法会阻塞住，直至数据加载完毕后通过notifyAll来唤醒
        // 如果loadFromDisk耗时过久，那么getString方法就会一直阻塞，从而导致卡顿
        awaitLoadedLocked();
        // 从内存map中依据key获取value
        String v = (String)mMap.get(key);
        return v != null ? v : defValue;
    }
}
```

```js
@GuardedBy("mLock")
private void awaitLoadedLocked() {
    // 是否加载完
    if (!mLoaded) {
        // Raise an explicit StrictMode onReadFromDisk for this
        // thread, since the real read will be in a different
        // thread and otherwise ignored by StrictMode.
        BlockGuard.getThreadPolicy().onReadFromDisk();
    }
    // 如果没有加载完，那么就一直阻塞在此处等待
    while (!mLoaded) {
        try {
            mLock.wait();
        } catch (InterruptedException unused) {
        }
    }
    if (mThrowable != null) {
        throw new IllegalStateException(mThrowable);
    }
}
```

当通过SharePreference查询数据的时候，会先检查此SP文件的xml数据是否已经全部解析完并加载到内存的Map中（mLoaded是否为true）。如果已经加载完成，那么就从内存map中读取值；如果还未加载完成，那么读取线程就会被挂起等待，直到数据加载完成后将此线程恢复，恢复之后再从内存map中读取值。可见，如果sp xml文件比较大，从而导致解析的时间以及加载到内存的时间比较长，那么就会导致查询数据、写数据等操作阻塞住，如果读写数据操作在UI线程，那么就会导致UI线程卡顿甚至ANR。

### 3.4. 获取Editor

要想插入数据，先要获取Editor对象，通过调用edit方法可以获取Editor对象，Editor的真正实现类型是EditorImpl类型。

```js
@Override
public Editor edit() {
    // TODO: remove the need to call awaitLoadedLocked() when
    // requesting an editor.  will require some work on the
    // Editor, but then we should be able to do:
    //
    //      context.getSharedPreferences(..).edit().putString(..).apply()
    //
    // ... all without blocking.
    synchronized (mLock) {
        awaitLoadedLocked();
    }
    // 每次调用edit方法都会创建一个新的EditorImpl对象
    return new EditorImpl();
}
```

```js
public final class EditorImpl implements Editor {
    private final Object mEditorLock = new Object();

    // key-value数据暂存
    @GuardedBy("mEditorLock")
    private final Map<String, Object> mModified = new HashMap<>();

	// 是否清除全部数据的标记位
    @GuardedBy("mEditorLock")
    private boolean mClear = false;
	...
}
```

EditorImpl 实现了Editor接口，提供put系列方法、apply方法、commit方法等。EditorImpl对象中也有一个Map，主要是提交修改的记录。每次执行edit的时候实际都会生成一个崭新的EditorImpl对象。因此如果频繁调用edit方法，就会频繁创建EditorImpl对象，相对应就会创建许多HashMap对象，默认HashMap的Size是16，虽然实际并不是很占用内存，但是确实没有必要这样去浪费。

### 3.5. 插入数据

提交新数据主要有两步：

1. 在内存中将EditorImpl中记录的修改和SharedPreferencesImpl中的原数据进行合并；
2. 将合并之后的数据写入xml文件。

```js
public final class EditorImpl implements Editor {
    private final Object mEditorLock = new Object();

    // key-value数据暂存
    @GuardedBy("mEditorLock")
    private final Map<String, Object> mModified = new HashMap<>();

	// 是否清除全部数据的标记位
    @GuardedBy("mEditorLock")
    private boolean mClear = false;

    @Override
    public Editor putString(String key, @Nullable String value) {
        synchronized (mEditorLock) {
            //插入数据, 先暂存到mModified对象
            mModified.put(key, value);
            return this;
        }
    }
   	...
    // 移除数据
    @Override
    public Editor remove(String key) {
        synchronized (mEditorLock) {
            mModified.put(key, this);
            return this;
        }
    }

    // 清空全部数据
    @Override
    public Editor clear() {
        synchronized (mEditorLock) {
            // 仅仅是改变mClear变量的状态，并未进行真正的清除
            mClear = true;
            return this;
        }
    }
	...
}
```

数据修改操作仅仅是修改mModified和mClear。直到数据提交commit或许apply过程， 才会真正的把数据更新到SharedPreferencesImpl(简称SPI)。比如设置mClear=true则会清空SPI的mMap数据。

### 3.6. commit 提交

```js
@Override
public boolean commit() {
    // commit开始的时间戳
    long startTime = 0;

    if (DEBUG) {
        startTime = System.currentTimeMillis();
    }
	// 在内存中将EditorImpl中记录的修改和SharedPreferencesImpl中的原数据进行合并,得到一个新的map，封装在MemoryCommitResult
    // 将数据更新到内存，将相关数据包装为一个MemoryCommitResult对象返回 
    MemoryCommitResult mcr = commitToMemory();
    // 将内存数据同步到文件，任务加入对象
    SharedPreferencesImpl.this.enqueueDiskWrite(
        mcr, null /* sync write on this thread okay */);
    try {
        // 进入等待状态, 直到写入文件的操作完成
        mcr.writtenToDiskLatch.await();
    } catch (InterruptedException e) {
        return false;
    } finally {
        if (DEBUG) {
            Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                    + " committed after " + (System.currentTimeMillis() - startTime)
                    + " ms");
        }
    }
    // 通知监听者, 并在主线程回调onSharedPreferenceChanged()方法
    notifyListeners(mcr);
    // 返回文件操作的结果数据
    return mcr.writeToDiskResult;
}
```

数据更新到内存：commitToMemory

```js
private MemoryCommitResult commitToMemory() {
    long memoryStateGeneration;
    boolean keysCleared = false;
    //保存发生变化的key列表
    List<String> keysModified = null;
    //外部监听器
    Set<OnSharedPreferenceChangeListener> listeners = null;
	// 最后要写入到磁盘xml文件的key-value map；初始值就是mMap
    Map<String, Object> mapToWriteToDisk;

    synchronized (SharedPreferencesImpl.this.mLock) {
        if (mDiskWritesInFlight > 0) {
            // 把SP中的Map内存拷贝一份作为内存修改的基准（初始状态）
            mMap = new HashMap<String, Object>(mMap);
        }
        //将成员mMap赋值给局部变量，后续for循环中用
        mapToWriteToDisk = mMap;
		mDiskWritesInFlight++;
		...

        synchronized (mEditorLock) {
            // 是否真正发生了改变。该标志主要作用是确保当前是否真正发生变化，避免无谓的I/O操作。
            boolean changesMade = false;
            // 全部清除的标记被设置为了true
            if (mClear) {
                if (!mapToWriteToDisk.isEmpty()) {
                    changesMade = true;
                    // 清除所有数据
                    mapToWriteToDisk.clear();
                }
                // keys要全部清除，标记为true
                keysCleared = true;
                mClear = false;
            }
            // 通过editor的所有put操作，都会先将数据存入mModified这个临时map中
            // 此处对put的所有key-value进行遍历
            for (Map.Entry<String, Object> e : mModified.entrySet()) {
                String k = e.getKey();
                Object v = e.getValue();
                // 当value值为this或null, 并且sp加载的map中存在此key，则移除相应的key;
                if (v == this || v == null) {
                    if (!mapToWriteToDisk.containsKey(k)) {
                        continue;
                    }
                    mapToWriteToDisk.remove(k);
                } else {
                    if (mapToWriteToDisk.containsKey(k)) {
                        Object existingValue = mapToWriteToDisk.get(k);
                        // value不为null并且没有改变，那么继续下一轮循环
                        if (existingValue != null && existingValue.equals(v)) {
                            // 如果value相等则跳过本次
                            // 主要是考虑changesMode标志位，确认当前数据是否真正发生变化
                            continue;
                        }
                    }
                    // value值发生改变了，那么将此新添加的key-value添加到sp map中；
                    mapToWriteToDisk.put(k, v);
                }
                // 在for循环中，如果发生数据变化，该changeMade将会置为true,表示当前数据发生变化
                changesMade = true;
				...
            }
            // 清空临时修改数据容器
            mModified.clear();
	...
	// 将5个参数包装成一个MemoryCommitResult对象返回
    return new MemoryCommitResult(memoryStateGeneration, keysCleared, keysModified,
            listeners, mapToWriteToDisk);
}
```

内存数据更新之后就会开始写入磁盘：enqueueDiskWrite。

```js
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
    // 根据postWriteRunnable是否为null判断执行此方法是commit方法调用，还是apply方法调用
    // 如果是commit，那么postWriteRunnable==null
    final boolean isFromSyncCommit = (postWriteRunnable == null);
    // 写入磁盘任务
    final Runnable writeToDiskRunnable = new Runnable() {
            @Override
            public void run() {
                // 等待获取写入磁盘的锁,一个SPI对象，无论commit还是apply的磁盘写操作是竞争同一把锁。
                synchronized (mWritingToDiskLock) {
                    // 获取到锁之后，将数据写入磁盘
					// 全量写入
                    writeToFile(mcr, isFromSyncCommit);
                }
                synchronized (mLock) {
                    mDiskWritesInFlight--; // 写入后减一
                }
				...
            }
        };

    // 当commit提交时，会在当前线程执行run方法
    if (isFromSyncCommit) {
        boolean wasEmpty = false;
        synchronized (mLock) {
            wasEmpty = mDiskWritesInFlight == 1;
        }
        if (wasEmpty) {
            // commit操作，直接在当前线程中执行;所以commit是同步操作
            writeToDiskRunnable.run();
            return;
        }
    }
	......
}
```

1. 先执行commitToMemory方法，在内存中将EditorImpl中记录的修改和SharedPreferencesImpl中的原数据进行合并，得到MemoryCommitResult；
2. 执行enqueueDiskWrite方法，在**当前线程**执行writeToDiskRunnable任务的run方法，将合并后的map数据全量写入到磁盘xml文件。**注意是当前线程执行写入磁盘的操作，也就是说commit可以在主线程执行，也可以在工作线程执行**。网上很多说commit是在UI线程执行，这是不正确的。

### 3.7. apply 异步提交

```js
public void apply() {
    final long startTime = System.currentTimeMillis();
    // 数据更新到内存，返回一个MemoryCommitResult对象
    final MemoryCommitResult mcr = commitToMemory();
    // 定义一个awaitCommit等待任务，CountDownLatch实现
    final Runnable awaitCommit = new Runnable() {
            @Override
            public void run() {
                try {
                    // 进入等待状态
                    mcr.writtenToDiskLatch.await();
                } catch (InterruptedException ignored) {
                }

                if (DEBUG && mcr.wasWritten) {
                    Log.d(TAG, mFile.getName() + ":" + mcr.memoryStateGeneration
                            + " applied after " + (System.currentTimeMillis() - startTime)
                            + " ms");
                }
            }
        };
    // 将awaitCommit任务添加到finisher任务队列，finisher任务主要是用于等待异步任务的处理
    QueuedWork.addFinisher(awaitCommit);

    Runnable postWriteRunnable = new Runnable() {
            @Override
            public void run() {
                // 进入等待状态，阻塞住，等待writeToFile磁盘写入完成后就移除awitCommit任务
                awaitCommit.run();
                QueuedWork.removeFinisher(awaitCommit);
            }
        };
    // 任务加入队列，异步执行磁盘写入
    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

	// 不等待结果
    notifyListeners(mcr);
}
```

```js
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
    // 根据postWriteRunnable是否为null判断执行此方法是commit方法调用，还是apply方法调用
    // 如果是apply，那么postWriteRunnable!=null isFromSyncCommit = false
    final boolean isFromSyncCommit = (postWriteRunnable == null);
    // 写入磁盘任务
    final Runnable writeToDiskRunnable = new Runnable() {
            @Override
            public void run() {
                // 等待获取写入磁盘的锁
                synchronized (mWritingToDiskLock) {
                    // 获取到锁之后，将数据写入磁盘
					// 全量写入
                    writeToFile(mcr, isFromSyncCommit);
                }
                synchronized (mLock) {
                    mDiskWritesInFlight--;
                }
                // 对于apply方法，走这里
                if (postWriteRunnable != null) {
                    postWriteRunnable.run();
                }
            }
        };
	...
    // 如果是apply(),提交则将任务加入线程池排队执行, 延迟100毫秒发送任务到队列
    // shouldDelay=!isFromSyncCommit
    QueuedWork.queue(writeToDiskRunnable, !isFromSyncCommit);
}
```

```js
public static void queue(Runnable work, boolean shouldDelay) {
    // 此Handler是一个子线程的Handler
    Handler handler = getHandler();

    synchronized (sLock) {
        // sWork是一个LinkedList队列，任务添加到对象
        sWork.add(work);
        // 延迟100毫秒通知执行任务
        if (shouldDelay && sCanDelay) {
            handler.sendEmptyMessageDelayed(QueuedWorkHandler.MSG_RUN, DELAY);
        } else {
            handler.sendEmptyMessage(QueuedWorkHandler.MSG_RUN);
        }
    }
}

// Handler是子线程的，优先级是前台线程优先级
private static Handler getHandler() {
    synchronized (sLock) {
        if (sHandler == null) {
            HandlerThread handlerThread = new HandlerThread("queued-work-looper",
                    Process.THREAD_PRIORITY_FOREGROUND);
            handlerThread.start();

            sHandler = new QueuedWorkHandler(handlerThread.getLooper());
        }
        return sHandler;
    }
}

private static class QueuedWorkHandler extends Handler {
    static final int MSG_RUN = 1;
	...
    public void handleMessage(Message msg) {
        if (msg.what == MSG_RUN) {
            processPendingWork();
        }
    }
}

private static void processPendingWork() {
	...
        if (work.size() > 0) {
            // 依次执行队列中的待处理任务的run方法
            for (Runnable w : work) {
                w.run();
            }
			....
        }
    }
}
```

1. 先执行commitToMemory方法，在内存中将EditorImpl中记录的修改和SharedPreferencesImpl中的原数据进行合并，得到MemoryCommitResult；
2. 添加一个awaitCommit任务到finisher队列中；
3. 执行enqueueDiskWrite方法，向一个工作线程的sWork队列中添加一个写入磁盘的任务writeToDiskRunnable；
4. 当磁盘写入任务执行后，就会执行写入后的任务postWriteRunnable，该任务主要是触发awaitCommit任务的执行，并在其执行后将其从finisher队列中移除，表示某次磁盘写入任务完成了。如果finisher队列不为空，说明还有些写入磁盘的任务没有完成。QueuedWork的finisher队列在这里存在的价值主要是用于在Stop Service, finish BroadcastReceiver过程用于判定是否处理完所有的异步SP操作。

	```js
	@Override
	public void handlePauseActivity(ActivityClientRecord r, boolean finished, boolean userLeaving,
	        int configChanges, PendingTransactionActions pendingActions, String reason) {
		....
	    // Make sure any pending writes are now committed. 等待未完成的sp写入任务完成，放置数据丢失
	    if (r.isPreHoneycomb()) {
	        QueuedWork.waitToFinish();
	    }
		....
	}
	```

## 四、SP多线程分析

SharePreference是多线程安全的，主要通过synchronize互斥同步锁来保证线程安全；下面仔细看下几种情况：

### 4.1. 多线程执行getSharePreference

1. 先从缓存中获取SPI对象，多个线程在同一把锁对象上互斥同步
2. 缓存中没有获取到SPI对象，也是new一个SPI对象

	```js
	synchronized (ContextImpl.class) {
		// 尝试从缓存中获取SharePreferenceImpl对象
		final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();
		sp = cache.get(file);
		// 缓存中没有
		if (sp == null) {
			....
	    	// 创建SharedPreferencesImpl实现类对象
	    	sp = new SharedPreferencesImpl(file, mode);
	    	// 对象放入缓存，返回SharePreference
	    	cache.put(file, sp);
	    	return sp;
		}
	}
	```

	可见，无论是从缓存获取、还是new一个SPI，多个线程并发调用getSharePreference时，将会进行互斥同步，因此同一个sp文件名，多个线程获取的SPI是同一个对象。

### 4.2. 多线程查询

```js
@Override
public long getLong(String key, long defValue) {
    synchronized (mLock) {
        awaitLoadedLocked();
        Long v = (Long)mMap.get(key);
        return v != null ? v : defValue;
    }
}
```

所有的数据查询方法都会在mLock对象上互斥同步，mLock对象是SPI的一个对象`private final Object mLock = new Object();` 因此多个线程在操作同一个SPI对象进行查询时，是竞争同一把对象锁mLock，从而保证读的安全性。**为啥读也要加锁？多线程读Map也是不安全的？**

### 4.3. 多线程执行edit

每个edit方法执行都会创建一个EditorImpl对象，因此多个线程调用edit方法，将会创建对应个数的EditorImpl对象。因此edit方法无须锁同步。

### 4.4. 多线程插入数据

```js
public Editor putString(String key, @Nullable String value) {
    synchronized (mEditorLock) {
        //插入数据, 先暂存到mModified对象
        mModified.put(key, value);
        return this;
    }
}
```

插入数据时必须先通过edit获取EditorImpl对象，如果多个线程都调用了edit方法获取EditorImpl对象，那么它们是不同的对象，mEditorLock是EditorImpl内部的对象，因此多个线程执行时竞争的不是同一把锁，它们之间无须保证线程安全，因为操作的mModified map都不是同一个，每个线程都操作一个单独的map，不会发生并发安全问题。

但若多个线程是用同一个EditorImpl对象，那么对应的mEditorLock就是同一把锁，因此会互斥同步来保证对mModified的安全操作。

### 4.5. 多线程执行commit

首先要知道commit是可以在工作线程执行的，而不是非要在UI线程执行。如果在UI线程执行commit，那么commit将会阻塞UI，如果是在工作线程执行，commit不会阻塞UI。既然commit可以在工作线程执行，那么并发执行commit提交数据时会怎样呢？安全吗？假设线程A、线程B同时执行commit方法：

假设线程A执行到commitToMemory方法，先获取到锁SharedPreferencesImpl.this.mLock：

```js
private MemoryCommitResult commitToMemory() {
	...
	// 即便多个线程用不同的EditorImpl对象调用commit方法，此处仍然是一把锁，因为SPI是一个
	synchronized (SharedPreferencesImpl.this.mLock) {
	...
}
```

那么线程B此时不能获取到锁，因此只能等到A释放锁。当A将新提交的数据更新到内存mapToWriteToDisk后，释放锁；线程B立即获取到锁并执行内存数据更新，而此时线程A已经开始执行enqueueDiskWrite方法进行磁盘写入。

```js
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
	...
    final Runnable writeToDiskRunnable = new Runnable() {
            @Override
            public void run() {
                // 等待获取写入磁盘的锁
                synchronized (mWritingToDiskLock) {
                    // 获取到锁之后，将数据写入磁盘
                    writeToFile(mcr, isFromSyncCommit);
                }
				...
            }
        };
	...
}
```

如果当前没有其他线程拿到mWritingToDiskLock锁，那么线程A将获取到mWritingToDiskLock锁，开始执行writeToFile方法，将内存数据全量写入到sp文件中。此时B线程更新完内存后，释放锁，并且也开始执行enqueueDiskWrite方法，可是线程B还拿不到磁盘写入的锁，因此就在此等待获取锁。当线程A磁盘写入完成后，回到commit方法，执行CountDownLatch的await方法。

当线程A执行完磁盘写入任务后，就会释放锁，并返回结果给commit方法；此时线程B立即获取到磁盘写入的锁，并开始写入磁盘，完成后，也返回结果。至此，线程A和线程B并发执行commit的流程就结束了。

##### 遗留问题：mcr.writtenToDiskLatch.await() 作用

```js
try {
    // 进入等待状态, 直到写入文件的操作完成
    mcr.writtenToDiskLatch.await();
} 
...
```

而CountDownLatch的countDown是何时执行的？  

```js
private static class MemoryCommitResult {
	...
	final CountDownLatch writtenToDiskLatch = new CountDownLatch(1);

    void setDiskWriteResult(boolean wasWritten, boolean result) {
        this.wasWritten = wasWritten;
        writeToDiskResult = result;
        writtenToDiskLatch.countDown();
    }
}
```

setDiskWriteResult方法是在writeToFile中调用的。为什么是先调用writeToFile，再调用mcr.writtenToDiskLatch.await(); 对于commit操作而言，这个CountDownLatch其实没有什么用，因为commit的writeToFile是在当前线程执行的，是同步的。磁盘写入完成后返回才会执行mcr.writtenToDiskLatch.await(); 而这个await在commit方法中是不会阻塞的。那么为什么还要写这段code？

总结下并发commit的过程是这样的：内存数据更新是互斥同步，写磁盘文件也是互斥同步。并发commit能提高效率吗？当然能，因为写入内存和写入磁盘是两把锁，一个线程写磁盘时，另外一个线程就可以开始更新内存数据。如果在单个线程多次commit，那么只能等前一个commit操作全部完成（内存数据更新+磁盘写入）后才能开始。

### 4.6. 多线程执行apply

apply与commit的内存数据更新过程是完全一样的。主要不同的是磁盘写入。apply的磁盘写入任务是放在一个子线程HandlerTrehad的任务队列排队执行的，而不是在当前线程执行。

假设线程A执行到commitToMemory方法，先获取到锁SharedPreferencesImpl.this.mLock：

```js
private MemoryCommitResult commitToMemory() {
	...
	// 即便多个线程用不同的EditorImpl对象调用commit方法，此处仍然是一把锁，因为SPI是一个
	synchronized (SharedPreferencesImpl.this.mLock) {
	...
}
```

那么线程B此时不能获取到锁，因此只能等到A释放锁。当A将新提交的数据更新到内存mapToWriteToDisk后，释放锁；线程B立即获取到锁并执行内存数据更新，而此时线程A已经开始执行enqueueDiskWrite方法，将writeToDiskRunnable任务添加到队列中，然后就返回了。其他线程如果也执行到enqueueDiskWrite，那么也会将任务添加到任务队列中排队执行。

```js
private void enqueueDiskWrite(final MemoryCommitResult mcr,
                              final Runnable postWriteRunnable) {
	...
    final Runnable writeToDiskRunnable = new Runnable() {
            @Override
            public void run() {
                // 等待获取写入磁盘的锁；注意对于同一个SPI，该锁是一把锁，部分commit还是apply
                synchronized (mWritingToDiskLock) {
                    // 获取到锁之后，将数据写入磁盘
                    writeToFile(mcr, isFromSyncCommit);
                }
				...
            }
        };
	...
	// 任务添加到HandlerThread线程的任务队列中
	QueuedWork.queue(writeToDiskRunnable, !isFromSyncCommit);
}
```

方法返回后，就直接通知监听器了，也就是apply方法完全不关注是否写入成功了。

```js
// 任务加入队列，异步执行磁盘写入
SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);

// Okay to notify the listeners before it's hit disk
// because the listeners should always get the same
// SharedPreferences instance back, which has the
// changes reflected in memory.
notifyListeners(mcr);
```

那么写入磁盘的任务怎么执行的：是按照任务队列串行执行任务的run方法，这些任务的run方法执行时，也必须先获取mWritingToDiskLock锁，因此它们的执行也是互斥同步，避免了多个线程同时操作磁盘IO带来安全问题。

**如果commit、apply本身就是在工作线程调用的情况下，apply与commit到底有何区别？apply新开辟的磁盘写入线程有什么意义呢？** 并发commit与并发apply看起来没有什么区别。

1. apply没有返回值, commit有boolean返回值能知道修改是否提交成功；
2. apply和commit都是将修改先更新到内存，再写入到磁盘文件; commit是在当前线程（不一定非要在UI线程）执行磁盘写入；而apply是在另一个工作线程执行磁盘写入；

##### 遗留问题：apply 也会导致OOM

SP写操作提交的过程分为两步：

1. 在内存中将EditorImpl中记录的修改和SharedPreferencesImpl中的原数据进行合并；
2. 将合并之后的数据写入xml文件。在第一步中如果同时存在多个apply请求，每个都会把SP中的Map内存拷贝一份作为内存修改的基准：

```js
if (mDiskWritesInFlight > 0) {
    ...
    mMap = new HashMap<String, Object>(mMap);
}
```

同时由于执行apply()时，第二步是在一个单线程线程池中执行的，因此第一步申请的内存实际上都被执行队列同时引用着。
由此我们可以知道，当反复执行SP的edit()和apply()操作时，假设操作的次数为M，而SP中的条目数量为N，HashMap.Entry对象最小占用24bytes，那么占用的总内存约为：`MN24*2(HashMap的容积率4/3到8/3之间，取平均数)`，举个例子，若是M=N=1024，则执行开头那个for循环时申请的内存为48M，这个大小已经足够一些app产生OOM了。经过这些分析，我们会有以下结论：

1. 如果只创建一个Editor并只进行一次apply()，不会有OOM。
2. 如果执行的不是apply()而是commit()，也不会有OOM，当然这样的性能会非常感人。
3. 循环执行edit()和apply()，达到1k数量级时，很容易引发OOM。在不同的机型上测试的数据会不同。虽然实际循环执行edit、apply的场景几乎很少，但是倘若真有批量的数据操作时，比如做数据迁移，那么就需要避免此种做法，而是利用一个Editor，批量put后，再执行一次apply。

### 4.7. 主线程（UI线程）多次commit、多次apply

一般情况下，App内对Sp的操作并不会放在工作线程去做。那么此种情况下commit与apply有什么区别呢？

1. 多次commit操作：一个commit操作包括更新内存数据、写磁盘IO，都是在UI线程完成；后面的commit要执行必须等前一个commit操作完成；如果时间比较久，就会导致UI卡顿甚至ANR；
2. 多次apply操作：一个commit操作包括更新内存数据、写磁盘IO；其中更新内存数据是在UI线程完成，而写磁盘IO是在另外一个工作线程执行，因此多次apply之后，内存更新很快，而磁盘IO写入的任务则会在单线程的任务队列中排队，依次写入磁盘。从而提高效率，不会阻塞UI。

```js
private void writeApply(int size) {
    // 1370ms 执行完返回，磁盘写入任务不知道何时完成的
    long sT = System.currentTimeMillis();
    for (int i = 0; i < size; i++) {
        SharedPreferences sharedPreferences = getSharedPreferences("jacky", MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putString("name" + i, "Jacky" + i);
        editor.putInt("age" + i, 18 + i);
        editor.apply();
    }
    Log.d(TAG, "write: span:" + (System.currentTimeMillis() - sT));
}

private void writeCommit(int size) {
    // 5220ms 才全部执行完返回
    long sT = System.currentTimeMillis();
    for (int i = 0; i < size; i++) {
        SharedPreferences sharedPreferences = getSharedPreferences("jacky", MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putString("name" + i, "Jacky" + i);
        editor.putInt("age" + i, 18 + i);
        editor.commit();
    }
    Log.d(TAG, "write: span:" + (System.currentTimeMillis() - sT));
}
```

如果上面两个线程在UI线程执行，那么显然commit方法阻塞UI的时间比apply方法长很多，主要是因为commit的磁盘IO操作是在当前线程执行，而apply是在工作线程执行的。如果将上面两个方法都放入子线程执行，那么都不会阻塞UI。

```js
new Thread(() -> writeApply(1500)).start();
new Thread(() -> writeCommit(1500)).start();
```

## 五、SharePreference的注意事项

1. 最好提前初始化SharedPreferences，避免SharedPreferences第一次创建时读取文件线程未结束而出现等待情况。除非你很确幸第一创建时读取文件到内存的耗时很低。
2. 强烈建议不要在sp里面存储特别大的key/value, 比如json string或xml string，否则会占用大量内存、或导致卡顿/anr；
3. sp配置不要全部都写在一个文件中，这样不仅第一次加载会很慢，也会占用大量内存。最好是**根据一定规则**分成多个sp文件。比如频繁和不频繁写入的配置就分别存储在两个不同的文件中。高频写操作的key与高频读操作的key可以适当地拆分文件, 可以减少同步锁竞争;
4. sp文件的写入是全量写入，即使改了一条配置，写入的时候也会对整个文件进行操作，因此最好能批量操作，不要每次都commit。
5. apply方法虽然是在线程中异步将配置写入文件，但是如果任务很多，而且每个任务执行时间很长，也可能会导致Activity或Service在stop的时候出现ANR。
6. 每次commit时会把全部的数据更新的文件, 所以整个文件是不应该过大的, 影响整体性能;
7. 不要连续多次edit()和apply, 应该获取一次获取edit(),然后多次执行putxxx(), 减少内存波动; 经常看到大家喜欢封装方法, 结果就导致这种情况的出现。大量循环调用edit和apply进行批量操作，可能会导致OOM。
8. 不要一上来就执行getSharedPreferences().edit(), 应该分成两大步骤来做, 中间可以执行其他代码；

## 六、封装SP工具类

需求分析：

1. 支持多文件名
2. 支持apply、commit两种提交方式
3. 支持批量写入
4. 不要频繁创建SharePreference实现类SharePreferenceImpl对象
5. 不要频繁调用edit方法创建Editor实现类EditorImpl对象

```js
package com.android.lib.tools.singleton;

import android.content.Context;
import android.content.SharedPreferences;
import android.preference.PreferenceManager;
import java.util.HashMap;

public class SPSingleton {
    private static volatile HashMap<String, SPSingleton> instanceMap = new HashMap<>();

    private SharedPreferences sharedPreferences;
    private SharedPreferences.Editor editor;

    //是否是执行apply的模式，false表示为commit保存数据
    private boolean isApplyMode = false;
    private static final String DEFAULT = "default";

    private SPSingleton(String name) {
        if (DEFAULT.equals(name)) {
            sharedPreferences = PreferenceManager.getDefaultSharedPreferences(APP.get());
        } else {
            sharedPreferences = APP.get().getSharedPreferences(name, Context.MODE_PRIVATE);
        }
        editor = sharedPreferences.edit();
    }

    public static SPSingleton get(String name) {
        if (instanceMap.get(name) == null) {
            synchronized (SPSingleton.class) {
                if (instanceMap.get(name) == null) {
                    instanceMap.put(name, new SPSingleton(name));
                }
            }
        }
        //这里每次get操作时强制将保存模式改为commit的方式
        instanceMap.get(name).isApplyMode = false;
        return instanceMap.get(name);
    }

    public static SPSingleton get() {
        return get(DEFAULT);
    }

    // 如果用apply模式的话，得要先调用这个方法，
    // 然后链式调用后续的存储方法，最后以commit方法结尾
    public SPSingleton applyMode() {
        isApplyMode = true;
        return this;
    }

    public void commit() {
        editor.commit();
    }

    public SPSingleton putBoolean(String key, boolean value) {
        editor.putBoolean(key, value);
        save();
        return this;
    }

    private void save() {
        if (isApplyMode) {
            editor.apply();
        } else {
            editor.commit();
        }
    }

    public SPSingleton putFloat(String key, float value) {
        editor.putFloat(key, value);
        save();
        return this;
    }

    public float getFloat(String key, float defValue){
        return sharedPreferences.getFloat(key, defValue);
    }

    public SPSingleton putLong(String key, long value) {
        editor.putLong(key, value);
        save();
        return this;
    }

    public long getLong(String key, long defValue){
        return sharedPreferences.getLong(key, defValue);
    }

    public SPSingleton putInt(String key, int value) {
        editor.putInt(key, value);
        save();
        return this;
    }

    public SPSingleton putString(String key, String value) {
        editor.putString(key, value);
        save();
        return this;
    }

    public String getString(String key, String defValue) {
        return sharedPreferences.getString(key, defValue);
    }

    public void delete(String key) {
        editor.remove(key);
        save();
    }

    public int getInt(String key, int defValue) {
        return sharedPreferences.getInt(key, defValue);
    }

    public boolean getBoolean(String key, boolean defValue) {
        return sharedPreferences.getBoolean(key, defValue);
    }

}
```





















