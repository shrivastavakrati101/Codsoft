# Codsoft// AlarmClockApp.kt
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.ViewModelProvider

class AlarmClockApp : AppCompatActivity() {
    private lateinit var viewModel: AlarmClockViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_alarm_clock)

        viewModel = ViewModelProvider(this).get(AlarmClockViewModel::class.java)

        // Initialize UI components
        val alarmList = findViewById<RecyclerView>(R.id.alarm_list)
        val addButton = findViewById<Button>(R.id.add_button)
        val settingsButton = findViewById<Button>(R.id.settings_button)

        // Set up alarm list adapter
        val adapter = AlarmListAdapter(viewModel.alarmList)
        alarmList.adapter = adapter

        // Set up add button click listener
        addButton.setOnClickListener {
            // Create a new alarm and add it to the list
            val newAlarm = Alarm("New Alarm", 8, 0, 0)
            viewModel.addAlarm(newAlarm)
            adapter.notifyDataSetChanged()
        }

        // Set up settings button click listener
        settingsButton.setOnClickListener {
            // Navigate to settings screen
            val intent = Intent(this, SettingsActivity::class.java)
            startActivity(intent)
        }
    }
}

// AlarmClockViewModel.kt
import android.app.AlarmManager
import android.content.Context
import androidx.lifecycle.ViewModel

class AlarmClockViewModel(private val context: Context) : ViewModel() {
    private val alarmManager = context.getSystemService(Context.ALARM_SERVICE) as AlarmManager
    val alarmList = mutableListOf<Alarm>()

    fun addAlarm(alarm: Alarm) {
        alarmList.add(alarm)
        // Set alarm using AlarmManager
        val intent = Intent(context, AlarmReceiver::class.java)
        val pendingIntent = PendingIntent.getBroadcast(context, 0, intent, 0)
        alarmManager.set(AlarmManager.RTC_WAKEUP, alarm.time, pendingIntent)
    }

    fun snoozeAlarm(alarm: Alarm) {
        // Snooze alarm for 10 minutes
        val newTime = alarm.time + 10 * 60 * 1000
        alarmManager.set(AlarmManager.RTC_WAKEUP, newTime, pendingIntent)
    }

    fun dismissAlarm(alarm: Alarm) {
        // Cancel alarm
        alarmManager.cancel(pendingIntent)
    }
}

// Alarm.kt
data class Alarm(val name: String, val hour: Int, val minute: Int, val second: Int) {
    val time: Long
        get() = hour * 60 * 60 * 1000 + minute * 60 * 1000 + second * 1000
}

// AlarmReceiver.kt
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent

class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Play alarm sound and show notification
        val notificationManager = context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        val notification = NotificationCompat.Builder(context, "alarm_channel")
            .setContentTitle("Alarm")
            .setContentText("Wake up!")
            .setSmallIcon(R.drawable.ic_alarm)
            .build()
        notificationManager.notify(1, notification)
    }
}

// activity_alarm_clock.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/alarm_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/add_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Add Alarm" />

    <Button
        android:id="@+id/settings_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Settings" />

</LinearLayout>

// AlarmListAdapter.kt
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView

class AlarmListAdapter(private val alarmList: List<Alarm>) : RecyclerView.Adapter<AlarmListAdapter.ViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_alarm, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val alarm = alarmList[position]
        holder.nameTextView.text = alarm.name
        holder.timeTextView.text = "${
