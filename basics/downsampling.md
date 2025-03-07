# Downsampling 

**Supported Capture Hardware:**\
❌ CW-Nano\
✅ CW-Lite\
✅ CW-Pro\
✅ CW-Husky

**Required ChipWhisperer software installation:**\
✅ any release

The ChipWhisperer's downsampling factor allows the ADC to discard a number of samples. For example, with a downsampling factor of 4, only every 4th sample is kept - the other 3 are thrown out before the trace is sent back to the computer. This feature has several uses:

* On the ChipWhisperer Lite, space is very limited - the maximum sample count of 24400 can make it difficult to find interesting features in traces or capture longer operations like ECC. Using downsampling can make it easier to find interesting features in these traces.
* On the ChipWhisperer Pro or Husky, the sampling rate in streaming mode is limited by the USB connection's data rate: the maximum sampling rate is approximately 10 MS/s (20 for Husky). Using downsampling can make it easier to fit under this limit. 

For all of these use cases, make sure you're already running the ADC clock slowly if possible. Don't use a 4x faster ADC just to throw out 3/4 of the samples! 

Downsampling is controlled via the `scope.adc.decimate` property.

```{caution}
Be careful using the ChipWhisperer Pro or Husky SAD trigger! The discarded ADC samples are still used as input for the SAD trigger module. If your reference trace was captured using downsampling, then the ChipWhisperer will never find this downsampled pattern in the raw samples.
```

## Example


The following shows the standard AES, captured on the CW-Lite at 1:128 downsampling (with a 29.538 MS/s ADC, giving effective sample rate of 230.77 KS/s):

![](../Images/Cwlite-longcapture.png)

This capture remains synchronous to the target device still, despite the huge downsampling factor. This allows attacks to succeed at surprisingly low sample rates (this sample rate is probably a little extreme, but you can see the 100 mS capture time).

