# Alarm Manger
안드로이드 생명주기 밖에서 시간 관리를 해준다.

## 1. Create a AlarmManager
```
AlarmManager alarmManager= (AlarmManager) context.getSystemService(ALARM_SERVICE);
```

## 2. Create Broadcast Receiver
BroadcastReceiver is what will be triggered when the Alarm goes off at the scheduled time

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "Alarm Triggered", Toast.LENGTH_SHORT).show();
    }
}
```

## 3. Create a PendingIntent
Creating a PendingIntent is the most important part of scheduling an Alarm using AlarmManager  
In fact this is what defines the operation which will be executed once the Alarm is triggered.  

#### What is Pending Intent ?

PendingIntent is an object which wraps another intent object.  
We pass the pending intent to the AlarmManager which basically gives AlarmManager  
the permission to trigger the wrapped intent as if it is being triggered from your own app(Even if your application is killed)

```java
public static final int REQUEST_CODE=101;
Intent intent=new Intent(this,MyReceiver.class);
PendingIntent.getBroadcast(this,REQUEST_CODE,intent,PendingIntent.FLAG_UPDATE_CURRENT);
/*
1st Param : Context
2nd Param : Integer request code
3rd Param : Wrapped Intent
4th Intent: Flag
*/
```

* Always keep a note of the request code used while creating a pending intent. Request code is a unique identifier of that pending intent and can be used to get reference to that PendingIntent anywhere inside the application.
* One more important factor while creating a pending intent is the flag passed in as the 4th parameter. Flags define the behavior of a PendingIntent and in turn the behavior of that Alarm.

1. **FLAG_CANCEL_CURRENT**
```
Flag indicating that if the described PendingIntent already exists, the current 
one should be canceled before generating a new one.

```
2. **FLAG_IMMUTABLE**
```
Flag indicating that the created PendingIntent should be immutable.
```

3. **FLAG_NO_CREATE**
```
Flag indicating that if the described PendingIntent does not already exist,
then simply return null instead of creating it.
```

4. **FLAG_ONE_SHOT**
```
Flag indicating that this PendingIntent can be used only once.
```

5. **FLAG_UPDATE_CURRENT**
```
Flag indicating that if the described PendingIntent already exists, then keep 
it but replace its extra data with what is in this new Intent.
```

**✋Not choosing a proper flag could result in multiple duplicate alarms scheduled around the same time**

## 4. Setting the Alarm
AlarmManager provides with number of methods which can be used based on how precise you want the alarm to be.
* setInexactRepeating  
    This schedules a repeating alarm which is inexact in its trigger time
    
    ```java
    AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
    Intent intent = new Intent(this, MyReceiver.class);
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, REQUEST_CODE, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    /*
    Alarm will be triggered approximately after one hour and will be repeated every hour after that
    */
    alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP, System.currentTimeMillis() + AlarmManager.INTERVAL_HOUR, AlarmManager.INTERVAL_HOUR, pendingIntent);
    /*
    1st Param : Type of the Alarm
    2nd Param : Time in milliseconds when the alarm will be triggered first
    3rd Param : Interval after which alarm will be repeated . You can only use any one of the AlarmManager constants
    4th Param :Pending Intent
    */
    ```

* setRepeating  
    This is same as setInExactRepeating except for the fact that the alarm is triggered exactly at the scheduled time.

    ```java
    AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
    Intent intent = new Intent(this, MyReceiver.class);
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, REQUEST_CODE, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(System.currentTimeMillis());
    calendar.set(Calendar.HOUR_OF_DAY, 8);
    calendar.set(Calendar.MINUTE, 30);
    /*
    Alarm will be triggered exactly at 8:30 every day
    */
    alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), AlarmManager.INTERVAL_DAY, pendingIntent);
    /*
    1st Param : Type of the Alarm
    2nd Parm : Time in milliseconds when the alarm will be triggered first
    3rd Param : Interval after which alarm will be repeated .
    4th Param : Pending Intent
    Note that we have changed the type to RTC_WAKEUP as we are using Wall clock time
    */
    ```

* set  
    This will schedule a one time alarm which will be triggered approximately at the scheduled time.

    ```java
    AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
    Intent intent = new Intent(this, MyReceiver.class);
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, REQUEST_CODE, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    //This alarm will trigger once approximately after 1 hour and
    alarmManager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, System.currentTimeMillis() + AlarmManager.INTERVAL_HOUR, pendingIntent);
    ```

* setExact  
    This shouldn’t be used unless there is a strong demand for alarm to be triggered at a precise time

    ```java

    AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
    
    Intent intent = new Intent(this, MyReceiver.class);
    
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, REQUEST_CODE, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    
    Calendar calendar = Calendar.getInstance();
    
    calendar.setTimeInMillis(System.currentTimeMillis());
    calendar.set(Calendar.HOUR_OF_DAY, 5);
    calendar.set(Calendar.MINUTE, 30);
    /*
    Alarm will be triggered once exactly at 5:30
    */
        alarmManager.setExact(AlarmManager.RTC, calendar.getTimeInMillis(), pendingIntent);
    /*
    1st Param : Type of the Alarm
    2nd Param : Time in milliseconds when the alarm will be triggered first
    3rd Param :Pending Intent
    Note that we have changed the type to RTC as we are using Wall clock time. Also device wont wake up
    when the alarm is triggered
    */
    ```

## Canceling the Alarm:
Canceling a scheduled alarm is much simpler than creating one.

```
alarmManager.cancel(pendingIntent);
```
