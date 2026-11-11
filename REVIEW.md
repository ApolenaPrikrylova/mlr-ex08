# REVIEW - Task 2f

Page: https://apolenaprikrylova.github.io/mlr-ex08/

1. It runs on my phone
- What I did: I opened the URL on my iPhone, allowed the camera and pointed it at a leaf.
- What I found: The page runs the model in the browser on the live rear camera and updates the probabilities continuously. In the screenshot it predicts apple with 97,2 %.
- What was fixed: Nothing

2. The same numbers as the notebook.
- What I did: I uploaded check_image.jpg on the page in Chrome and in Safari on my Mac and compared the result with check_expected.json. The agent also ran the check on the deployed page in headless Chrome.
class		check_expected.json	page (Chrome)	page (Safari)
apple		0.5703			0.5703		0.477
cherry		0.3542			0.3542		0.440
chestnut	0.0210			0.0210		0.022
maple		0.0544			0.0544		0.061

- What I found:In chrome the numbers agree to three decimals. In safari they don't. The cause is safari fingerprinting protection, which adds small random noise to pixels read from a canvas. The page reads the image this way, so the model gets slightly changed input. The preprocessing and the model are correct, as Chrome shows.
- What was fixed: Nothing in the code, bc a page cannot turn this protection off. The iPhone also uses safari, so live results on the phone can be a bit noisier than in the notebook.

3. The preprocessing matches the model
- What I did: Looked at preprocess() in index.htm (lines 86-99) and compared it with model_info.json and the notebook.
- What I found:
Line 90: the central square, const s = Math.min(sw, sh);
Line 92: crop and resize to 96 × 96 in one step, ctx.drawImage(src, (sw - s) / 2, (sh - s) / 2, s, s, 0, 0, n, n);
Line 97: divide by 255 and standardize (d[i * 4 + c] / 255 - info.mean[c]) / info.std[c]
Channel order: R, G, B, each channel in its own plane (CHW). The alpha channel is dropped.
Line 98: the tensor has shape [1, 3, 96, 96].
The input size, mean and std are read from model_info.json: 96, mean [0.485, 0.456, 0.406], std [0.229, 0.224, 0.225].

The notebook uses ToTensor() (divides by 255, HWC → CHW) and Normalize(mean, std) with the same values. The photos in the teacher's ZIP were already center-cropped and resized to 96x96, so the page does the same steps on the camera frame. The only small difference is that the browser's resizing is not exactly the same as PIL's.
- What was fixed: Nothing 

4. Nothing leaves the phone
- What I did: I searched index.html for every network call and checked which URLs the page loads.
- What I found: The image is not sent anywhere. There is no XMLHttpRequest, WebSocket, form or sendBeacon. Fetch is only used for the page'sown files. Camera frames and uploaded photos are drawn to a canvas and classified in the browser.
URLs the page loads:
URL											why
cdn.jsdelivr.net/npm/onnxruntime-web@1.20.1/dist/ort.min.js (+ its .wasm file)		ONNX Runtime Web, runs the model in the browser
cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4/qrcode.min.js			draws the QR code at the bottom of the page
model_info.json, leaf_model.onnx (my GitHub Pages)					the model and its preprocessing
check_image.jpg, check_expected.json (my GitHub Pages)					only for the self-check

The CDNs only see the visitor's IP address and browser, never the image.
- What was fixed: Nothing
