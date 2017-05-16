# LayoutInflater使用中重点关注inflate方法的参数含义 #：

1. inflate(xmlId, null); 只创建temp的View，然后直接返回temp。

1. inflate(xmlId, parent); 创建temp的View，然后执行root.addView(temp, params);最后返回root。

1. inflate(xmlId, parent, false); 创建temp的View，然后执行temp.setLayoutParams(params);然后再返回temp。

1. inflate(xmlId, parent, true); 创建temp的View，然后执行root.addView(temp, params);最后返回root。

1. inflate(xmlId, null, false); 只创建temp的View，然后直接返回temp。

1. inflate(xmlId, null, true); 只创建temp的View，然后直接返回temp。

Activity.setContentView()方法实际就是把设置的view add到id为content的framelayout上
所以setContentView()可以去掉改为

LayoutInflater.from(Context).inflate(ResourceId,parent)

或LayoutInflater.from(Context).inflate(ResourceId,parent，true)

或View myView = LayoutInflater.from(Context).inflate(ResourceId,parent，false);
findViewById(android.R.id.content).addView(myView,myView.getLayoutParams);