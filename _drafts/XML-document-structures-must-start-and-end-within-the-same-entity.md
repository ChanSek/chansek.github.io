XML document structures must start and end within the same entity.

Problem - 
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:gravity="center_vertical"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:textAppearance="@style/TextAppearance.AppCompat.Medium"
    tools:text="Pan india">

Solution -
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:gravity="center_vertical"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:textAppearance="@style/TextAppearance.AppCompat.Medium"
    tools:text="Pan india" />


Conclusion - We should always close the XML tag before starting a new tag.
