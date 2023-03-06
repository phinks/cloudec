# cloudec

## A cloud-based codec for streaming real time video.

Cloudec is the first codec with an integrated network stack. Cloudec assumes that there is an unreliable network between the encoder and decoder, and it tries to apply different encoding techniques to different parts of the source image in order to create the best experience for the end user whilst also optimizing network use. Unlike other codecs, which produce one output stream or blob for each input, cloudec will produce a sequence of images and manage their lifetime on the network until they are successfully decoded and displayed by the decoder; and it dynamically changes its output in response to detected network conditions. If part of an image is lost, it can decide whether it is necessary to re-send that part or if there is another update already on the wire that makes that redundant. It can decide what minimal amount of re-sending is required to replace any lost image. It can re-encode an image (even at a different quality) with minimal cost. It will also have a number of techniques to minimise the effects of network conditions.

### Traditional streaming stack.

In most real time video stacks (I have worked on several implementations of online meetings and virtual desktops), a video source is fed into a black box encoder, which spits out a blob of compressed image. This is then passed to a network stack which manages the sending and delivery of that blob. On the receive side that blob is re-assembled and then fed into another black box decoder, which outputs an approximation of the source image to be rendered in the client. 
We should one here that the encoder is fed a single image which will all be encoded with the same parameters - a single quality for example regardless of any characteristics that may be detectable in the image and could be used to encode a part at a lower quality because it may be transient, or another at a higher quality because it contains something important like text. There may be ways round this by doing some pre-processing of the image before encoding, but this is expensive and will probably duplicate much of the effort that the encoder is going to do anyway, albeit it without providing the results in any useful way.
In this use case, the encoder can be informed of network conditions - a low bandwidth network can lead to a reduced frame rate, or lower quality images for example. But that feedback only affects the latest image - there may be several in the queue waiting to be written to the network which cannot be changed. 
The real problem with this mode, though, is loss recovery. Because we have no idea what the encoder generated, which parts of the image it actually encoded, and similarly for any images yet to be processed, if the decoder does not receive the (whole or part of) blob, all it can do is generate a nack which will probably cause the encoder to encode the whole image and re-send it, regardless of how much of that image was actually lost. If the cause of this situation was a degraded network, this is absolutely the worst response since it puts more pressure on the network, so encouraging more loss and therefore causing more keyframes to be requested. 

### Cloudec steaming stack.

In the case of cloudec, the source image is fed into the encoder as before, but from then on the network delivery, receipt and decoding are handled entirely internally. Because of the tight coupling of the encoder and the network, the image can be encoded optimally for the instantaneous network conditions when the imager is being processed, not those that were detected a few frames ago.
Cloudec has several (configurable) options to maximize network efficiency. It analyses images across frames and will only try to send those parts that have changed since the last encode. It can determine if a part of an image is likely to be transient, and encode that at a lower quality. It can determine if a part of an image is not transient, or important (eg text) and encode that at a higher quality. It can also re-encode low quality images at a higher quality when bandwidth allows.

Because cloudec keeps track of all of the state of all images it has processed until they are decoded and rendered, it can optimize loss recovery. If a frame is lost, cloudec can determine if any re-send is necessary (if a later frame contains content for the same regions, recovery is a no-op). If any re-send is necessary, cloudec can generate a minimum required region, again at very low cost, and send only that.


## Network optimization techniques.

Dynamic quality adjustment. Cloudec will analyze its workload and determine which regions are high importance. Low importance regions can be more highly compressed. 

Phasing. On lossy networks images may be split into 2 or 4 sub-images on a checkerboard pattern and sent separately. If less than the full compliment of sub-images are received the decoder will interpolate the missing pieces.

Scrolling. If part of the image is moved horizontally or vertically, the decoder will use its existing copy of the image and move it on the destination, saving the case of re-sending the same data.

Image cacheing. Similar images will be read from a cache rather than re-encoded and re-sent.




