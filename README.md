# 📱 Lab 12 — Google Maps & Localisation (Android Java)

## 🎯 Objectif

* Afficher une Google Map dans l’application
* Demander la permission de localisation
* Écouter les changements de position (Network/GPS)
* Ajouter un marker à chaque nouvelle position
* Gérer le GPS désactivé (boîte de dialogue)
* Zoomer sur la position

---

## 🧱 Partie 1 : Création du projet

### Étape 1 — Création

* Android Studio → New Project
* Choisir **Google Maps Activity**

### Étape 2 — Structure générée

* `MapsActivity.java`
* `activity_maps.xml`
* `google_maps_api.xml`
* Dépendances Google Maps

---

## 🔑 Partie 2 : Clé API Google Maps

### Configuration

```xml
<string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">
    VOTRE_CLE_ICI
</string>
```

---

## 🔐 Partie 3 : Permissions

### AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
```

---

## ⚠️ Gestion GPS désactivé

```java
private void buildAlertMessageNoGps() {
    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setMessage("Your GPS seems to be disabled, do you want to enable it?")
            .setCancelable(false)
            .setPositiveButton("Yes", new DialogInterface.OnClickListener() {
                public void onClick(final DialogInterface dialog, final int id) {
                    startActivity(new Intent(android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS));
                }
            })
            .setNegativeButton("No", new DialogInterface.OnClickListener() {
                public void onClick(final DialogInterface dialog, final int id) {
                    dialog.cancel();
                }
            });
    final AlertDialog alert = builder.create();
    alert.show();
}
```

---

## 🗺️ Partie 4 : onMapReady()

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mMap = googleMap;

    LocationManager locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);

    LatLng sydney = new LatLng(-34, 151);
    mMap.addMarker(new MarkerOptions().position(sydney).title("Marker in Sydney"));

    boolean permissionGranted =
            ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
                    == PackageManager.PERMISSION_GRANTED;

    if (permissionGranted) {

        locationManager.requestLocationUpdates(
                LocationManager.NETWORK_PROVIDER,
                1000,
                50,
                new LocationListener() {

                    @Override
                    public void onLocationChanged(Location location) {
                        double latitude = location.getLatitude();
                        double longitude = location.getLongitude();

                        LatLng position = new LatLng(latitude, longitude);
                        mMap.addMarker(new MarkerOptions().position(position).title("Marker"));

                        float zoomLevel = 15.0f;
                        mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(position, zoomLevel));
                    }

                    @Override
                    public void onStatusChanged(String provider, int status, Bundle extras) {}

                    @Override
                    public void onProviderEnabled(String provider) {}

                    @Override
                    public void onProviderDisabled(String provider) {
                        buildAlertMessageNoGps();
                    }
                }
        );

    } else {
        ActivityCompat.requestPermissions(
                this,
                new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                200
        );
    }

    mMap.moveCamera(CameraUpdateFactory.newLatLng(sydney));
}
```

---

## 🔁 Partie 5 : Permissions Runtime

```java
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);

    if (requestCode == 200) {
        if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            onMapReady(mMap);
        }
    }
}
```

---

## 🎯 Partie 6 : Marker unique (optimisé)

```java
private Marker currentMarker;

@Override
public void onLocationChanged(Location location) {
    LatLng pos = new LatLng(location.getLatitude(), location.getLongitude());

    if (currentMarker == null) {
        currentMarker = mMap.addMarker(new MarkerOptions().position(pos).title("Position actuelle"));
    } else {
        currentMarker.setPosition(pos);
    }

    mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(pos, 15f));
}
```

---

## ⚡ Bonnes pratiques

* Utiliser `animateCamera` pour une meilleure UX
* Réduire `minDistance` pour les tests
* Vérifier que GPS est activé
* Éviter trop de markers

---

## ❗ Problèmes fréquents

* Carte blanche → clé API incorrecte
* Pas de position → permission refusée
* Pas de mise à jour → distance trop élevée

---

## ✅ Résultat final

L’application :

* Affiche une carte Google Maps
* Récupère la position utilisateur
* Ajoute un marker dynamique
* Centre et zoom automatiquement
* Gère les permissions et le GPS
