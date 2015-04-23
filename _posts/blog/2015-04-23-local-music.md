---
layout: post
keywords: blog
description: blog
title: "本地音乐查询播放Demo"
categories: [Android]
tags: [Media,Android,数据库,UI]
---
{% include JB/setup %}
##截屏
![shot](http://7xiqgb.com1.z0.glb.clouddn.com/shot.png)

##[项目GitHub地址](https://github.com/JingHaifeng/LocalAudioDemo)

##Overview
####难点
 * UI对象动态更新
 * CursorAdapter与LoaderManager的配合
 * MediaPlayer的使用


##数据来源
>**MediaStore**
>
>The Media provider contains meta data for all available media on both internal and external storage devices.

实现`LoaderManager.LoaderCallbacks<Cursor>`，绑定数据给适配器

    private static final String[] COLUMNS = {
            MediaStore.Audio.Media._ID,
            MediaStore.Audio.Media.TITLE,
            MediaStore.Audio.Media.ARTIST,
            MediaStore.Audio.Media.DURATION,
            MediaStore.Audio.Media.ALBUM,
            MediaStore.Audio.Media.DATA,
            MediaStore.Audio.Media.SIZE,
    };

    @Override
    public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {
        CursorLoader cursorLoader = new CursorLoader(getActivity(),
                MediaStore.Audio.Media.EXTERNAL_CONTENT_URI,
                COLUMNS,
                MediaStore.Audio.Media.IS_MUSIC + "!=?",
                new String[]{"0"},
                MediaStore.Audio.Media.TITLE + " ASC");
        return cursorLoader;
    }

    @Override
    public void onLoadFinished(Loader<Cursor> cursorLoader, Cursor cursor) {
        audioCursorAdapter.changeCursor(cursor);
    }

    @Override
    public void onLoaderReset(Loader<Cursor> cursorLoader) {
        audioCursorAdapter.changeCursor(null);
    }

##数据适配
>**CursorAdapter**
>
>Static library support version of the framework's CursorAdapter. Used to write apps that run on platforms prior to Android 3.0. When running on Android 3.0 or above, this implementation is still used; it does not try to switch to the framework's implementation. See the framework SDK documentation for a class overview.

CursorAdapter在3.0之后版本加入API，使用V4包可适配2.2之后版本。CursorAdapter大大简化与数据库数据与UI的绑定过程，自动的数据库操作，避免人为的数据库泄漏。

必须实现的三个方法

    @Override
    public AudioData getItem(int position) {

        return fromCursor(getCursor(), position);
    }

    @Override
    public View newView(Context context, Cursor cursor, ViewGroup viewGroup) {
        return layoutInflater.inflate(R.layout.audio_item_layout, null);
    }

    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        final int position = cursor.getPosition();
        final Holder holder = getHolder(view);
        final AudioData audioData = getItem(position);
        holder.title.setText(audioData.getTitle());
        holder.artist.setText(audioData.getArtist());
        	 holder.title.setTextColor(mContext.getResources().getColor(R.color.grey));
        holder.playLayout.setVisibility(View.GONE);
        holder.currentTime.setText(CommonUtil.millisTimeToDotFormat(0, false, false));
        final long duration = audioData.getDuration();
        if (duration > 0) {
            holder.duration.setText(CommonUtil.millisTimeToDotFormat(duration, false, false));    
        }

    }

##事件监听、媒体播放
有两个个事件需要监听

1. item click
2. list scroll

点击后MediaPlayer状态：播放、暂停、切换

滚动时，动态刷新标题、播放时间等UI对象，item的状体能正常显示。

    private AdapterView.OnItemClickListener onItemClickListener = new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, final View view, final int position, long id) {
                if (curPostion == position) {
                    if (mediaPlayer.isPlaying()) {
                        mediaPlayer.pause();
                        pauseState();
                        if (handler.hasMessages(FRESH_CUR_TIME)){
                            handler.removeMessages(FRESH_CUR_TIME);
                        }
                    } else {
                        mediaPlayer.start();
                        showPlayView(view);
                        handler.sendEmptyMessage(FRESH_CUR_TIME);
                    }
                } else {
                    pauseState();
                    mediaPlayer.reset();
                    try {
                        mediaPlayer.setDataSource(audioCursorAdapter.getItem(position).getUrl());
                        mediaPlayer.prepareAsync();
                        mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                            @Override
                            public void onPrepared(MediaPlayer mp) {
                                mediaPlayer.start();
                                showPlayView(view);
                                handler.sendEmptyMessage(FRESH_CUR_TIME);
                            }
                        });
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                curPostion = position;
            }
    };


            @Override //滚动事件
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                if (curPostion < 0) {
                    return;
                }
                if (curPostion >= firstVisibleItem && curPostion <= firstVisibleItem + visibleItemCount - 1) {
                    if (curPostion == firstVisibleItem) {
                        showPlayView(view.getChildAt(0));
                    } else if (curPostion == firstVisibleItem + visibleItemCount - 1) {
                        showPlayView(view.getChildAt(visibleItemCount - 1));
                    }
                } else {
                    titleTv = null;
                    curTimeTv = null;
                    playLayout = null;
                }

            }

##UI刷新
每隔100ms查询播放状态，刷新时间显示

 	@Override
    public void handleMessage(Message msg) {
	handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    if (mediaPlayer.isPlaying()) {
                        curTime = mediaPlayer.getCurrentPosition();
                        if (curTimeTv != null) {
                            curTimeTv.setText(CommonUtil.millisTimeToDotFormat(curTime,false,false));
                        }
                        handler.sendEmptyMessage(FRESH_CUR_TIME);
                    } else {
                        handler.removeMessages(FRESH_CUR_TIME);
                        curTime = 0;
                    }
                }
            }, 100);
        }

