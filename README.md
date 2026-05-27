📡 Receiver Demo (BroadcastReceiver Android)

⸻

🟢 Étape 1 — Création du projet

🔧 1.1 Projet Android

* Android Studio → New Project
* Template : Empty Activity
* Nom : ReceiverDemo
* Language : Java
* Min SDK : API 24
* Finish

⸻

🟡 Étape 2 — Receiver Dynamique (Mode Avion)

📁 AirplaneModeReceiver.java

package com.example.receiverdemo;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;
public class AirplaneModeReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_AIRPLANE_MODE_CHANGED.equals(intent.getAction())) {
            boolean isAirplaneOn = intent.getBooleanExtra("state", false);
            String message = isAirplaneOn
                    ? "Mode Avion ACTIVÉ - Plus de connexion !"
                    : "Mode Avion DÉSACTIVÉ - Connexion rétablie";
            Toast.makeText(context, message, Toast.LENGTH_LONG).show();
        }
    }
}

📌 Explication rapide

* BroadcastReceiver → écoute les événements système
* onReceive() → exécuté automatiquement
* ACTION_AIRPLANE_MODE_CHANGED → broadcast système
* getBooleanExtra("state") → état du mode avion

⸻

🔵 Étape 3 — Receiver Statique (BOOT_COMPLETED)

📁 BootReceiver.java

package com.example.receiverdemo;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;
public class BootReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            Toast.makeText(context,
                    "Téléphone démarré - Receiver statique activé !",
                    Toast.LENGTH_LONG).show();
        }
    }
}

⸻

🟠 Étape 4 — Manifest

📌 Permission

<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

📌 Receivers

<application>
    <receiver
        android:name=".BootReceiver"
        android:exported="false">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
    </receiver>
    <receiver
        android:name=".CustomEventReceiver"
        android:exported="false"/>
</application>

📌 Explication

* BOOT_COMPLETED → fonctionne même sans lancer l’app
* exported=false → sécurité Android 12+
* Receiver statique = déclaré dans le Manifest

⸻

🔵 Étape 5 — MainActivity (Receiver dynamique + custom broadcast)

package com.example.receiverdemo;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
public class MainActivity extends AppCompatActivity {
    private AirplaneModeReceiver airplaneReceiver;
    private boolean isReceiverRegistered = false;
    private Button btnToggleAirplane, btnSendCustom;
    private TextView tvStatus;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        airplaneReceiver = new AirplaneModeReceiver();
        tvStatus = findViewById(R.id.tvStatus);
        btnToggleAirplane = findViewById(R.id.btnToggleAirplane);
        btnSendCustom = findViewById(R.id.btnSendCustom);
        btnToggleAirplane.setOnClickListener(v -> toggleAirplaneReceiver());
        btnSendCustom.setOnClickListener(v -> sendCustomBroadcast());
    }
    private void toggleAirplaneReceiver() {
        if (!isReceiverRegistered) {
            IntentFilter filter = new IntentFilter();
            filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED);
            registerReceiver(airplaneReceiver, filter);
            isReceiverRegistered = true;
            tvStatus.setText("Receiver Mode Avion : ACTIVÉ");
            btnToggleAirplane.setText("Désactiver Receiver");
        } else {
            unregisterReceiver(airplaneReceiver);
            isReceiverRegistered = false;
            tvStatus.setText("Receiver Mode Avion : DÉSACTIVÉ");
            btnToggleAirplane.setText("Activer Receiver");
        }
    }
    private void sendCustomBroadcast() {
        Intent intent = new Intent("com.example.receiverdemo.CUSTOM_EVENT");
        intent.putExtra("message", "Bonjour depuis custom broadcast !");
        sendBroadcast(intent);
        Toast.makeText(this,
                "Custom Broadcast envoyé !",
                Toast.LENGTH_SHORT).show();
    }
    @Override
    protected void onDestroy() {
        if (isReceiverRegistered) {
            unregisterReceiver(airplaneReceiver);
        }
        super.onDestroy();
    }
}

⸻

🟣 Étape 6 — Custom Receiver

package com.example.receiverdemo;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;
public class CustomEventReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if ("com.example.receiverdemo.CUSTOM_EVENT".equals(intent.getAction())) {
            String message = intent.getStringExtra("message");
            Toast.makeText(context,
                    "Custom reçu : " + message,
                    Toast.LENGTH_LONG).show();
        }
    }
}

⸻

🟢 Étape 7 — Layout

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">
    <TextView
        android:id="@+id/tvStatus"
        android:text="Status"
        android:textSize="18sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <Button
        android:id="@+id/btnToggleAirplane"
        android:text="Activer Receiver Avion"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <Button
        android:id="@+id/btnSendCustom"
        android:text="Envoyer Custom Broadcast"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>

⸻

🧪 Étape 8 — Test

✔️ Tests à faire

* Activer receiver → changer mode avion → Toast
* Envoyer custom broadcast → Toast immédiat
* Redémarrer émulateur → BootReceiver déclenché
