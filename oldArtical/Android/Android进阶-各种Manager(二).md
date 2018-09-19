title: Android项目-各种Manager(二)
date: 2016/2/29 15:42:29   
categories: Android
---

# Android项目-各种Manager(二) #

## TelephonyManager ##
>- Provides access to information about the telephony services on the device. Applications can use the methods in this class 
>- to determine telephony services and states, as well as to access some types of subscriber information. Applications can also
>- register a listener to receive notification of telephony state changes. 
>- 当获取一些隐私信息时当然需要权限的啦。

使用范例：

	/*1. 监听电话状态*/
	tm = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
	listener = new MyPhoneListener();
	tm.listen(listener, PhoneStateListener.LISTEN_CALL_STATE);
	
	//我们的监听器
	private class MyPhoneListener extends PhoneStateListener {
	@Override
	public void onCallStateChanged(int state, String incomingNumber) {
		super.onCallStateChanged(state, incomingNumber);
		switch (state) {
		case TelephonyManager.CALL_STATE_IDLE:// 空闲状态
			/*...*/
			break;
		case TelephonyManager.CALL_STATE_RINGING:// 响铃状态
			break;
		case TelephonyManager.CALL_STATE_OFFHOOK:// 接通状态
			break;
		}
	}

	/*获取和SIM可有关信息*/
	tm.getSimSerialNumber()；
    tm.getSimState()；
	.....
	/*获取和NetWork有关的信息*/

## DevicePolicyManager ##
