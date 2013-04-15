---
layout: post
title: "ViewPager &amp; the verticality of Fragments"
date: 2012-01-12 19:31:00
comments: true
categories: [Android, Old Enki Blog]
---
{% img right http://blog.renaud.io/images/verticality.png %}

<span itemprop="description">Back to august 2010, [Rich Hyndman](https://plus.google.com/115995639636688350464/posts) posts an [entry on the android developers' blog](http://android-developers.blogspot.com/2011/08/horizontal-view-swiping-with-viewpager.html) entitled "Horizontal View Swiping with ViewPager". I already was really enthusiastic about the Android Support Package at that time because of the introduction of the LoaderManager which save us big headaches on the proper cursor management (but that's another story). In an application of mine, I've used an onFling/swipe gesture based on some 2009 codes found on the Web ([codeshogun.com](http://www.codeshogun.com/blog/2009/04/16/how-to-implement-swipe-action-in-android/) and/or [ceveni.com](http://www.ceveni.com/2009/08/android-gestures-detection-sample-code.html) for example).</span>

The main drawbacks on my onFling approach and implementation are:

1. That is based on a threshold velocity
2. Once initiated, the animation goes on and can't be reversed
3. The gesture is the trigger of a startActivity call and each of those activities are… activities!

Surely my onFling implementation was too dumb and since I read Rich's post, I have the secret personal project to use the ViewPager and one of the underlying adapters to reimplement that ugly onFling part of our application.

Well, the time has come with the Christmas holidays!

The point is that my onFling-based activity is now composed by two fragment and the FragmentPagerAdapter is nice but manage only one Fragment as a ViewPager current page content.

Now, what if a want a vertical stack of several fragments? ⇒ Read and adapt the FragmentPagerAdapter source code from the Support Package (compact version without comments):

{% codeblock lang:java %}
public abstract class VerticalFragmentsPagerAdapter extends PagerAdapter {
    private static final String TAG = "VerticalFragmentsPagerAdapter";
    private static final boolean DEBUG = false;
    private final FragmentManager mFragmentManager;
    private final int mRows;
    private FragmentTransaction mCurTransaction = null;
    private ArrayList<Fragment> mCurrentPrimaryItem = null;

    public VerticalFragmentsPagerAdapter(FragmentManager fm, int rows) {
        mFragmentManager = fm;
        mRows = rows;
        mCurrentPrimaryItem = new ArrayList<Fragment>(mRows);
    }

    /**
     * Return the Fragments associated with a specified position.
     */
    public abstract ArrayList<Fragment> getItem(int position);

    @Override
    public void startUpdate(ViewGroup container) {}

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }

        Context ctx = container.getContext();
        LayoutInflater inflater =
                (LayoutInflater) ctx.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        View layout = inflater.inflate(R.layout.pager, null);
        layout.setId(6109 + position);
        LinearLayout ll = (LinearLayout) layout.findViewById(R.id.pager_linear_layout);
        ll.setId(900913 + position);
        ((ViewPager) container).addView(layout);
        Fragment firstFragment =
                mFragmentManager.findFragmentByTag(
                        makeFragmentName(container.getId(), position, 0));
        ArrayList<Fragment> frags = null;

        if (firstFragment != null) {
            frags = new ArrayList<Fragment>(mRows);

            for (int i = 0; i < mRows; ++i) {
                frags.add(mFragmentManager.findFragmentByTag(
                        makeFragmentName(container.getId(), position, i)));
            }

            for (Fragment f : frags) {
                if (DEBUG) Log.v(TAG, "Attaching item #" + position + ": f=" + f);
                mCurTransaction.attach(f);
            }
        } else {
            frags = getItem(position);
            int currentRow = 0;

            for (Fragment f : frags) {
                if (DEBUG) Log.v(TAG, "instanciateItem / Adding item #" + position + ": f= " + f);
                mCurTransaction.add(ll.getId(), f,
                        makeFragmentName(container.getId(), position, currentRow));
                currentRow++;
            }
        }

        if (frags != mCurrentPrimaryItem) {
            for (Fragment f : frags) {
                f.setMenuVisibility(false);
                f.setUserVisibleHint(false);
            }
        }

        return layout;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }

        for (int i = 0; i < mRows; ++i) {
            if (DEBUG) Log.v(TAG, "destroyItem / Detaching item #" + position + ": row=" + i);
            mCurTransaction.detach(
                    mFragmentManager.findFragmentByTag(
                            makeFragmentName(container.getId(), position, i)));
        }
        ((ViewPager) container).removeView((View)object);
    }

    @Override
    public void setPrimaryItem(ViewGroup container, int position, Object object) {
        ArrayList<Fragment> frags = new ArrayList<Fragment>(mRows);

        for (int i = 0; i < mRows; ++i) {
            frags.add(mFragmentManager.findFragmentByTag(
                    makeFragmentName(container.getId(), position, i)));
        }

        if (!frags.equals(mCurrentPrimaryItem)) {
            if (!mCurrentPrimaryItem.isEmpty() && (mCurrentPrimaryItem.get(0) != null)) {
                for (Fragment f : mCurrentPrimaryItem) {
                    f.setMenuVisibility(false);
                    f.setUserVisibleHint(false);
                }
            }

            if (!frags.isEmpty() && (frags.get(0) != null)) {
                for (Fragment f : frags) {
                    f.setMenuVisibility(true);
                    f.setUserVisibleHint(true);
                }
            }
            mCurrentPrimaryItem = (ArrayList<Fragment>) frags.clone();
            if (DEBUG) Log.v(TAG, "setPrimaryItem #" + position + ": " + mCurrentPrimaryItem);
        }
    }

    @Override
    public void finishUpdate(ViewGroup container) {
        if (mCurTransaction != null) {
            mCurTransaction.commitAllowingStateLoss();
            mCurTransaction = null;
            mFragmentManager.executePendingTransactions();
        }
    }

    @Override
    public boolean isViewFromObject(View view, Object object) { return view == ((View) object); }

    @Override
    public Parcelable saveState() { return null; }

    @Override
    public void restoreState(Parcelable state, ClassLoader loader) {}

    public static String makeFragmentName(int viewId, int index, int subindex) {
        return "android:switcher:" + viewId + ":" + index + ":" + subindex;
    }
}
{% endcodeblock %}

This implementation uses ArrayList as the fragments' container. The trick here is that instead of returning a fragment from instanciateItem, I return a  containing a LinearLayout.

Be warned: don't use several instances of the same fragment for a given position! That adapter is not designed to do that…

From the above code, it's now trivial to figure out how to extends that adapter: "Uh… the `getItem` should return an ArrayList of Fragments?" − "Yup!"

A trivial example is available on [github](https://github.com/nibua-r/org.summo.blog.verticality). Just clone and run `ant debug && ant installd`… Feel free to fork, improve and return your feedback.
