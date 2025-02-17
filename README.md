#AssemblyAdapter

这是Android上的一个通用Adapter库，有了它你就不需要再写新的Adapter了，并且还自带加载更多功能。支持BaseAdapter、BaseExpandableListAdapter以及RecyclerView.Adapter

This is a generic Android library on the Adapter, with it you do not need to write a new Adapter, and also comes loaded with more features. Supports BaseAdapter, BaseExpandableListAdapter, and RecyclerView.Adapter

##Features
>* 你再也不需要写新的Adapter了，只需为每一个item layout写一个AssemblyItemFactory即可，并且遵循AssemblyItemFactory结构的代码逻辑会前所未有的清晰
>* 可以一次使用多个AssemblyItemFactory，每个AssemblyItemFactory就代表一种itemType
>* 自带加载更多功能，你只需实现一个专门用于加载更多的尾巴即可
>* 支持BaseAdapter、BaseExpandableListAdapter以及RecyclerView.Adapter，已经涵盖了Android开发中用到的所有Adapter
>* 没有使用任何反射相关的代码无须担心性能问题
>* AssemblyItemFactory的扩展性很好，可以满足你的任何需求

English（From Baidu translate）
>* You don't need to write a new Adapter again, just for each layout AssemblyItemFactory write a item can, and follow the AssemblyItemFactory structure of the code logic will never clear
>* You can use more than one AssemblyItemFactory, each AssemblyItemFactory on behalf of a kind of itemType
>* Comes with more features, you just need to implement a dedicated to load more of the tail
>Supports BaseAdapter, BaseExpandableListAdapter, and RecyclerView.Adapter, has covered all the Android development of Adapter
>* There is no need to worry about performance issues without using any reflection related code
>* AssemblyItemFactory is very good and can meet any of your needs.

##Usage Guide
####1. 导入AssemblyAdapter（Import AssemblyAdapter to your project）

#####使用Gradle（Use Gradle）
``从JCenter仓库导入（Import from jcenter）``

```groovy
dependencies{
	compile 'me.xiaopan:assemblyadapter:1.0.0'
}
```

``离线模式（Offline work）``

首先到[releases](https://github.com/xiaopansky/AssemblyAdapter/releases)页面下载最新版的aar包（`这里以assemblyadapter-1.0.0.aar为例，具体请以你下载到的文件名称为准`），并放到你module的libs目录下

然后在你module的build.gradle文件中添加以下代码：
```groovy
repositories{
    flatDir(){
        dirs 'libs'
    }
}

dependencies{
    compile(name:'assemblyadapter-1.0.0', ext:'aar')
}
```
最后同步一下Gradle即可

#####使用Eclipse（Use Eclipse）
1. 首先到[releases](https://github.com/xiaopansky/AssemblyAdapter/releases)页面下载最新版的aar包（`这里以assemblyadapter-1.0.0.aar为例，具体请以你下载到的文件名称为准`）
2. 然后改后缀名为zip并解压
2. 接下来将classes.jar文件重命名为assemblyadapter-1.0.0.jar
3. 最后将assemblyadapter-1.0.0.jar拷贝到你的项目的libs目录下

####2. 配置最低版本（Configure min sdk version）
AssemblyAdapter最低兼容API v7

#####使用Gradle（Use Gradle）
在app/build.gradle文件文件中配置最低版本为7
```groovy
android {
	...

    defaultConfig {
        minSdkVersion 7
        ...
    }
}
```

#####使用Eclipse（Use Eclipse）
在AndroidManifest.xml文件中配置最低版本为7
```xml
<manifest
	...
	>
    <uses-sdk android:minSdkVersion="7"/>
    <application>
    ...
    </application>
</manifest>
```

###3. 使用AssemblyAdapter
``这里只讲解AssemblyAdapter的用法，AssemblyExpandableAdapter和AssemblyRecyclerAdapter的用法跟AssemblyAdapter完全一致，只是Factory和Item所继承的类不一样而已，详情请参考sample源码``

AssemblyAdapter分为三部分：
>* AssemblyAdapter：封装了Adapter部分需要的所有处理
>* AssemblyItemFactory：负责匹配Bean、创建AssemblyItem以及管理扩展参数
>* AssemblyItem：负责创建convertView、设置并处理事件、设置数据

AssemblyAdapter最大的特点如其名字所表达的意思是可以组装的，你可以调用其addItemFactory(AssemblyItemFactory)方法添加多个AssemblyItemFactory，而每一个AssemblyItemFactory就代表一种ItemType，这样就轻松实现了ItemType。因而AssemblyAdapter的数据列表的类型是Object的。

首先创建你的AssemblyItemFactory和AssemblyItem，如下所示：
```java
public class UserListItemFactory extends AssemblyItemFactory<UserListItemFactory.UserListItem> {
    private EventListener eventListener;

    public UserListItemFactory(Context context) {
        this.eventListener = new EventProcessor(context);
    }

    @Override
    public Class<?> getBeanClass() {
        return User.class;
    }

    @Override
    public UserListItem createAssemblyItem(ViewGroup parent) {
        return new UserListItem(parent, this);
    }

    public static class UserListItem extends AssemblyItem<User, UserListItemFactory> {
        private ImageView headImageView;
        private TextView nameTextView;
        private TextView sexTextView;
        private TextView ageTextView;
        private TextView jobTextView;

        protected UserListItem(ViewGroup parent, UserListItemFactory factory) {
            super(LayoutInflater.from(parent.getContext()).inflate(R.layout.list_item_user, parent, false), factory);
        }

        @Override
        protected void onFindViews(View convertView) {
	        // ... 在这里各种findViewById
            headImageView = (ImageView) convertView.findViewById(R.id.image_userListItem_head);
            nameTextView = (TextView) convertView.findViewById(R.id.text_userListItem_name);
            sexTextView = (TextView) convertView.findViewById(R.id.text_userListItem_sex);
            ageTextView = (TextView) convertView.findViewById(R.id.text_userListItem_age);
            jobTextView = (TextView) convertView.findViewById(R.id.text_userListItem_job);
        }

        @Override
        protected void onConfigViews(Context context) {
            // ... 你可以在这里注册一些点击事件并根据需要设置View的大小
            headImageView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    getItemFactory().eventListener.onClickHead(getPosition(), getData());
                }
            });
        }

        @Override
        protected void onSetData(int position, User user) {
	        // ... 在这里填充数据
            headImageView.setImageResource(user.headResId);
            nameTextView.setText(user.name);
            sexTextView.setText(user.sex);
            ageTextView.setText(user.age);
            jobTextView.setText(user.job);
        }
    }

    public interface EventListener{
        void onClickHead(int position, User user);
    }

    private static class EventProcessor implements EventListener {
        private Context context;

        public EventProcessor(Context context) {
            this.context = context;
        }

        @Override
        public void onClickHead(int position, User user) {
            Toast.makeText(context, "别摸我头，讨厌啦！", Toast.LENGTH_SHORT).show();
        }
    }
}
```
在创建AssemblyItemFactory和AssemblyItem的时候需要注意以下几点：
>* AssemblyItemFactory和AssemblyItem的泛型是互相指定的，一定要好好配置，不可省略
>* AssemblyItemFactory.getBeanClass()返回的Class是用来匹配数据源中的Bean的，因此要跟你传给AssemblyAdapter的数据源中包含的Bean类型一致
>* AssemblyItemFactory.createAssemblyItem(ViewGroup)方法返回的类型跟你在AssemblyItemFactory上配置的泛型一样
>* AssemblyItem的onFindViews(View)和onConfigViews(Context)只会在创建AssemblyItem的调用一次，而onSetData()则会在每次getView()的时候都调用
>* 你可以通过getPosition()和getData()方法直接获取当前Item所对应的位置和数据对象，因此你在处理onClick的时候不需要再通过setTag来传递数据了，直接调方法获取即可。getData()返回的对象类型和你在AssemblyItem第一个泛型配置的一样，因此你也不需要再转换类型了
>* 你可以像示例那样将所有需要处理的事件通过接口剥离出去，然后写一个专门的处理类，这样逻辑比较清晰

然后new一个AssemblyAdapter直接通过其addItemFactory方法使用你自定义的UserListItemFactory即可，如下：
```java
ListView listView = ...;
List<Object> dataList = ...;
dataList.add(new User(...));

AssemblyAdapter adapter = new AssemblyAdapter(dataList);
adapter.addItemFactory(new UserListItemFactory(getBaseContext()));
listView.setAdapter(adapter);
```

**一次使用多个AssemblyItemFactory，就像这样**

```java
ListView listView = ...;
List<Object> dataList = ...;
dataList.add(new User(...));
dataList.add(new Game(...));

AssemblyAdapter adapter = new AssemblyAdapter(dataList);
adapter.addItemFactory(new UserListItemFactory(getBaseContext()));
adapter.addItemFactory(new GameListItemFactory(getBaseContext()));
listView.setAdapter(adapter);
```

**使用加载更多功能**

首先你需要创建一个继承自AbstractLoadMoreListItemFactory的ItemFactory，AbstractLoadMoreListItemFactory已经将逻辑部分的代码写好了，你只需关心界面即可，如下：
```java
public class LoadMoreListItemFactory extends AbstractLoadMoreListItemFactory {

    public LoadMoreListItemFactory(EventListener eventListener) {
        super(eventListener);
    }

    @Override
    public AbstractLoadMoreListItem createAssemblyItem(ViewGroup parent) {
        return new LoadMoreListItem(parent, this);
    }

    public static class LoadMoreListItem extends AbstractLoadMoreListItem {
        private View loadingView;
        private View errorView;

        protected LoadMoreListItem(ViewGroup parent, AbstractLoadMoreListItemFactory baseFactory) {
            super(LayoutInflater.from(parent.getContext()).inflate(R.layout.list_item_load_more, parent, false), baseFactory);
        }

        @Override
        protected void onFindViews(View convertView) {
            loadingView = convertView.findViewById(R.id.text_loadMoreListItem_loading);
            errorView = convertView.findViewById(R.id.text_loadMoreListItem_error);
        }

        @Override
        public void showErrorRetry() {
            loadingView.setVisibility(View.GONE);
            errorView.setVisibility(View.VISIBLE);
        }

        @Override
        public void showLoading() {
            loadingView.setVisibility(View.VISIBLE);
            errorView.setVisibility(View.GONE);
        }

        @Override
        public View getErrorRetryView() {
            return errorView;
        }
    }
}
```
然后调用enableLoadMore(AbstractLoadMoreListItemFactory)方法开启，如下：
```java
AssemblyAdapter adapter = ...;
adapter.enableLoadMore(new LoadMoreListItemFactory(new AbstractLoadMoreListItemFactory.EventListener(){
	@Override
    public void onLoadMore(AbstractLoadMoreListItemFactory.AdapterCallback adapterCallback) {
        // ... 访问网络加载更多数据，最后调用adapterCallback或adapter的loadMoreFinished()方法结束加载
		adapterCallback.loadMoreFinished();
    }
}));
```
注意：
>* 加载完成了你需要调用adapterCallback.loadMoreFinished()方法结束加载
>* 如果加载失败了你需要调用adapterCallback.loadMoreFailed()方法，调用此方法后会自动调用LoadMoreListItem的showErrorRetry()方法显示失败提示，并且点击错误提示可以重新触发onLoadMore()方法
>* 如果没有更多数据可以加载了你就调用adapterCallback.disableLoadMore()方法关闭加载更多功能

除此之外还有AssemblyExpandableAdapter和AssemblyRecyclerAdapter分别用在ExpandableListView和RecyclerView，其用法和和AssemblyAdapter完全一样，只是ItemFactory和Item所继承的类不一样而已，详情请参考其[sample](https://github.com/xiaopansky/AssemblyAdapter/tree/master/sample)源码

##License
```java
/*
* Copyright (C) 2015 Peng fei Pan <sky@xiaopan.me>
*
* Licensed under the Apache License, Version 2.0 (the "License");
* you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
*
*   http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/
```
