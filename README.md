# Industrial-Training-PROJECT CODE


package com.tirthik.medopedia.activities;

import android.animation.ArgbEvaluator;
import android.animation.ValueAnimator;
import android.annotation.SuppressLint;
import android.content.DialogInterface;
import android.content.Intent;
import android.graphics.Color;
import android.os.Build;
import android.os.Bundle;
import android.support.design.widget.AppBarLayout;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.KeyEvent;
import android.view.Menu;
import android.view.MenuItem;
import android.view.MotionEvent;
import android.view.View;
import android.view.Window;
import android.view.inputmethod.EditorInfo;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.aurelhubert.ahbottomnavigation.AHBottomNavigation;
import com.aurelhubert.ahbottomnavigation.AHBottomNavigationAdapter;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.sarthak.medopedia.R;
import com.sarthak.medopedia.db.DBHelper;
import com.sarthak.medopedia.fragments.HistoryFragment;
import com.sarthak.medopedia.fragments.HomeFragment;
import com.sarthak.medopedia.fragments.MyMedicinesFragment;

public class MainActivity extends AppCompatActivity {

    EditText editTextSearch;
    DBHelper dbHelper;
    private AppBarLayout appBarLayout;
    private int previousColor = -1;
    private int currentFragmentPosition = 1;
//    FirebaseAuth firebaseAuth;
//    FirebaseUser firebaseUser;

    @SuppressLint("ClickableViewAccessibility")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

//        firebaseAuth=FirebaseAuth.getInstance();


//        firebaseUser=firebaseAuth.getCurrentUser();
      /*  if (firebaseUser==null){
            startActivity(new Intent(MainActivity.this,LoginActivity.class));
        }*/


        Toolbar toolbar = findViewById(R.id.toolbar);
        toolbar.setTitleTextColor(Color.WHITE);
        setSupportActionBar(toolbar);
        dbHelper = new DBHelper(this);

        appBarLayout = findViewById(R.id.app_bar_layout);

        editTextSearch = findViewById(R.id.edit_text_search);
        editTextSearch.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
                try {
                    if (actionId == EditorInfo.IME_ACTION_SEARCH) {
                        Intent intent = new Intent(MainActivity.this, SearchActivity.class);
                        intent.putExtra("search", editTextSearch.getText().toString());
                        startActivity(intent);
                        return true;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return false;
            }
        });
        editTextSearch.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                final int DRAWABLE_RIGHT = 2;

                if (event.getAction() == MotionEvent.ACTION_UP) {
                    if (event.getRawX() >= (editTextSearch.getRight() - editTextSearch.getCompoundDrawables()[DRAWABLE_RIGHT].getBounds().width())) {
                        try {
                            Intent intent = new Intent(MainActivity.this, SearchActivity.class);
                            intent.putExtra("search", editTextSearch.getText().toString());
                            startActivity(intent);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                        return true;
                    }
                }
                return false;
            }
        });

        AHBottomNavigation bottomNavigation = findViewById(R.id.bottom_navigation);

        int[] tabColors = new int[]{
                ContextCompat.getColor(getApplicationContext(), R.color.teal),
                ContextCompat.getColor(getApplicationContext(), R.color.red),
                ContextCompat.getColor(getApplicationContext(), R.color.blue)
        };
        AHBottomNavigationAdapter navigationAdapter = new AHBottomNavigationAdapter(this, R.menu.bottom_navigation_menu);
        navigationAdapter.setupWithBottomNavigation(bottomNavigation, tabColors);

        // Force to tint the drawable (useful for font with icon for example)+
        bottomNavigation.setForceTint(true);

        // Manage titles
        bottomNavigation.setTitleState(AHBottomNavigation.TitleState.ALWAYS_SHOW);

        // Use colored navigation with circle reveal effect
        bottomNavigation.setColored(true);

        // Set current item programmatically
        bottomNavigation.setCurrentItem(1);

        // Set listeners
        bottomNavigation.setOnTabSelectedListener(new AHBottomNavigation.OnTabSelectedListener() {
            @Override
            public boolean onTabSelected(int position, boolean wasSelected) {
                setFragment(position, false);
                changeColors(position);
                return true;
            }
        });

        setFragment(1, true);
        changeColors(1);
    }

    @Override
    public void onResume() {
        super.onResume();

        if (currentFragmentPosition == 2) {
            setFragment(2, true);
        }
    }

    private void changeColors(int position) {
        if (previousColor == -1) {
            previousColor = ContextCompat.getColor(getApplicationContext(), R.color.teal);
        }
        int newColor = Color.BLACK;
        switch (position) {
            case 0: {
                newColor = ContextCompat.getColor(getApplicationContext(), R.color.teal);
                break;
            }
            case 1: {
                newColor = ContextCompat.getColor(getApplicationContext(), R.color.red);
                break;
            }
            case 2: {
                newColor = ContextCompat.getColor(getApplicationContext(), R.color.blue);
                break;
            }
        }

        ValueAnimator animator = new ValueAnimator();
        animator.setIntValues(previousColor, newColor);
        animator.setEvaluator(new ArgbEvaluator());
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                if (appBarLayout != null) {
                    appBarLayout.setBackgroundColor((int) valueAnimator.getAnimatedValue());
                }
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    Window window = getWindow();
                    window.setStatusBarColor((int) valueAnimator.getAnimatedValue());
                }
            }
        });
        animator.setDuration(300);
        animator.start();

        previousColor = newColor;
    }

    public void setFragment(int position, boolean force) {
        if (currentFragmentPosition != position || force) {
            Fragment fragment;

            switch (position) {
                case 0: {
                    fragment = new HomeFragment();
                    break;
                }
                case 1: {
                    fragment = new MyMedicinesFragment();
                    break;
                }
                case 2: {
                    fragment = new HistoryFragment();
                    break;
                }
                default: {
                    fragment = new HomeFragment();
                    break;
                }
            }

            FragmentManager fragmentManager = getSupportFragmentManager();
            FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
            fragmentTransaction.replace(R.id.fragment_container, fragment);
            fragmentTransaction.commit();

            currentFragmentPosition = position;
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.menu_signout:
                showDialog();
                break;
        }
        return true;
    }

    private void showDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setIcon(R.drawable.ic_lock_open_black_24dp);
        builder.setTitle("Logout");
        builder.setMessage("Are you sure?");
        builder.setCancelable(false);
        builder.setPositiveButton("Yes", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                //firebaseAuth.signOut();
                Intent intent = new Intent(MainActivity.this, LoginActivity.class);
                startActivity(intent);
                Toast.makeText(getApplicationContext(), "Logged out successfully", 


Toast.LENGTH_LONG).show();
                finish();
            }
        });
        builder.setNegativeButton("No", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                    dialogInterface.dismiss();
            }
        });
        AlertDialog alertDialog = builder.create();
        alertDialog.show();
    }
}










