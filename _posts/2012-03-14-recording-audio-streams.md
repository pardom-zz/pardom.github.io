---
layout: post
title:  "Recording audio streams"
date:   2012-03-14 00:00:00
categories: android
summary: "My most recent app RingDimmer relies heavily on the use of the device microphone. This is not a trivial operation as there are many devices with many specifications, many of which don’t conform to the minimum requirements set out in the Android docs."
---
My most recent app [RingDimmer](http://www.ringdimmer.com/) relies heavily on the use of the device microphone. This is not a trivial operation as there are many devices with many specifications, many of which don’t conform to the [minimum requirements](https://developer.android.com/reference/android/media/AudioRecord.html) set out in the Android docs.

As such, I’ve decided to lend my fellow developers a hand and release the class I created to handle audio recording. I’m sure the class will need to improve over time, as new device configurations appear. So far it has been pretty reliable, but I welcome your comments and critiques.

{% highlight java %}
import android.media.AudioFormat;
import android.media.AudioRecord;
import android.media.MediaRecorder.AudioSource;
import android.os.Process;

public class AudioMeter extends Thread {
	/////////////////////////////////////////////////////////////////
	// PUBLIC CONSTANTS

	// Convenience constants
	public static final int AMP_SILENCE = 0;
	public static final int AMP_NORMAL_BREATHING = 10;
	public static final int AMP_MOSQUITO = 20;
	public static final int AMP_WHISPER = 30;
	public static final int AMP_STREAM = 40;
	public static final int AMP_QUIET_OFFICE = 50;
	public static final int AMP_NORMAL_CONVERSATION = 60;
	public static final int AMP_HAIR_DRYER = 70;
	public static final int AMP_GARBAGE_DISPOSAL = 80;

	/////////////////////////////////////////////////////////////////
	// PRIVATE CONSTANTS

	private static final float MAX_REPORTABLE_AMP = 32767f;
	private static final float MAX_REPORTABLE_DB = 90.3087f;

	/////////////////////////////////////////////////////////////////
	// PRIVATE MEMBERS

	private AudioRecord mAudioRecord;
	private int mSampleRate;
	private short mAudioFormat;
	private short mChannelConfig;

	private short[] mBuffer;
	private int mBufferSize = AudioRecord.ERROR_BAD_VALUE;

	private int mLocks = 0;

	/////////////////////////////////////////////////////////////////
	// CONSTRUCTOR

	private AudioMeter() {
		Process.setThreadPriority(Process.THREAD_PRIORITY_URGENT_AUDIO);
		createAudioRecord();
	}

	/////////////////////////////////////////////////////////////////
	// PUBLIC METHODS

	public static AudioMeter getInstance() {
		return InstanceHolder.INSTANCE;
	}

	public float getAmplitude() {
		return (float) (MAX_REPORTABLE_DB + (20 * Math.log10(getRawAmplitude() / MAX_REPORTABLE_AMP)));
	}

	public synchronized void startRecording() {
		if (mAudioRecord == null || mAudioRecord.getState() != AudioRecord.STATE_INITIALIZED) {
			throw new IllegalStateException("startRecording() called on an uninitialized AudioRecord.");
		}

		if (mLocks == 0) {
			mAudioRecord.startRecording();
		}

		mLocks++;
	}

	public synchronized void stopRecording() {
		mLocks--;

		if (mLocks == 0) {
			if (mAudioRecord != null) {
				mAudioRecord.stop();
				mAudioRecord.release();
				mAudioRecord = null;
			}
		}
	}

	/////////////////////////////////////////////////////////////////
	// PRIVATE METHODS

	private void createAudioRecord() {
		if (mSampleRate > 0 && mAudioFormat > 0 && mChannelConfig > 0) {
			mAudioRecord = new AudioRecord(AudioSource.MIC, mSampleRate, mChannelConfig, mAudioFormat, mBufferSize);
			return;
		}

		// Find best/compatible AudioRecord
		for (int sampleRate : new int[] { 8000, 11025, 16000, 22050, 32000, 44100, 47250, 48000 }) {
			for (short audioFormat : new short[] { AudioFormat.ENCODING_PCM_16BIT, AudioFormat.ENCODING_PCM_8BIT }) {
				for (short channelConfig : new short[] { AudioFormat.CHANNEL_IN_MONO, AudioFormat.CHANNEL_IN_STEREO,
					AudioFormat.CHANNEL_CONFIGURATION_MONO, AudioFormat.CHANNEL_CONFIGURATION_STEREO }) {

					// Try to initialize
					try {
						mBufferSize = AudioRecord.getMinBufferSize(sampleRate, channelConfig, audioFormat);

						if (mBufferSize < 0) {
							continue;
						}

						mBuffer = new short[mBufferSize];
						mAudioRecord = new AudioRecord(AudioSource.MIC, sampleRate, channelConfig, audioFormat,	mBufferSize);

						if (mAudioRecord.getState() == AudioRecord.STATE_INITIALIZED) {
							mSampleRate = sampleRate;
							mAudioFormat = audioFormat;
							mChannelConfig = channelConfig;

							return;
						}

						mAudioRecord.release();
						mAudioRecord = null;
					}
					catch (Exception e) {
						// Do nothing
					}
				}
			}
		}
	}

	private int getRawAmplitude() {
		if (mAudioRecord == null) {
			createAudioRecord();
		}

		final int bufferReadSize = mAudioRecord.read(mBuffer, 0, mBufferSize);

		if (bufferReadSize < 0) {
			return 0;
		}

		int sum = 0;
		for (int i = 0; i < bufferReadSize; i++) {
			sum += Math.abs(mBuffer[i]);
		}

		if (bufferReadSize > 0) {
			return sum / bufferReadSize;
		}

		return 0;
	}

	/////////////////////////////////////////////////////////////////
	// PRIVATE CLASSES

	private static class InstanceHolder {
		private static final AudioMeter INSTANCE = new AudioMeter();
	}
}
{% endhighlight %}

Basically what this class does is construct a valid AudioRecord Object, wrap AudioRecord methods, and provide conversion to decibels. It also caches the AudioRecord configuration and prevents multiple instances of the recorder.

Feel free to use as you see fit, and be sure to check out [RingDimmer](http://www.ringdimmer.com/)!
