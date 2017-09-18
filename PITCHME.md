# REST in peace

---

# WHOAMI

<ul>
  <li>University of Twente
    <ul>
      <li>BSc. Technische Informatica</li>
      <li>MSC. Computer Science (Kerckhoff's Institute)</li>
    </ul>
  </li>
  <li>Software Architect at Extendas
    <ul>
    	<li>Backend (PHP, Python)</li>
  	<li>Mobile (Swift)</li>
    </ul>
  </li>
</ul>

---

# App Development

<ul>
  <li>
    User Interface
    <ul>
      <li></li>
      <li></li>
    </ul>
  </li>
  <li>
    Storage
    <ul>
    	<li></li>
  	    <li></li>
    </ul>
  </li>
  <li>
    Networking
    <ul>
        <li>
    </ul>
  </li>
  <li>
    Diagnostics &amp; Logging
  </li>
</ul>


+++

# Storing data in Shared Preferences

```
public class Calc extends Activity {
    public static final String PREFS_NAME = "MyPrefsFile";

    @Override
    protected void onCreate(Bundle state){
       super.onCreate(state);
       . . .

       // Restore preferences
       SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
       boolean silent = settings.getBoolean("silentMode", false);
       setSilent(silent);
    }

    @Override
    protected void onStop(){
       super.onStop();

      // We need an Editor object to make preference changes.
      // All objects are from android.context.Context
      SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
      SharedPreferences.Editor editor = settings.edit();
      editor.putBoolean("silentMode", mSilentMode);

      // Commit the edits!
      editor.commit();
    }
}

```
