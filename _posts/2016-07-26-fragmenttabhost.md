---
layout: post
title: FragmentTabHost
tags:
- ui
- FragmentTabHost
categories: ui
description: FragmentTabHost切换Fragment时避免UI重新加载
---
FragmentTabHost切换Fragment时避免UI重新加载.  
用FragmentTabHost + Fragment,每次FragmentTabHost切换fragment时会调用onCreateView()重绘UI。 

{% highlight java %}  

    private View rootView;
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        if (rootView == null) {
            rootView = inflater.inflate(R.layout.fragment_crosswalk, null);
            ButterKnife.bind(this, rootView);

            initData();
            initView();
            // 读取数据
            mXwalkview.load(mURL, null);
        }
        // 缓存的rootView需要判断是否已经被加过parent，如果有parent需要从parent删除，要不然会发生这个rootview已经有parent的错误。
        ViewGroup parent = (ViewGroup) rootView.getParent();
        if (parent != null) {
            parent.removeView(rootView);
        }


        return rootView;
    }
{% endhighlight %}