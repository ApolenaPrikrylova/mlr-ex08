# REPORT - Task 4: A better classifier

Setup:
- Baseline: my Task 3 network (small CNN trained from scratch), Adam with lr 1e-3, 15 epochs, no augmentation.
- One change at a time against the baseline, then the best combinations. All runs use the same training images and fixed seed. The important rows were repeated with seeds 0, 1 and 2 (mean +- std).
- Three test sets:
val: our group's validation photos
camera-like val: the same images randomly cropped, rotated and colour-changed. They were generated once with a fixed seed, so every variant sees the same images.
old val: last year's validation photos (120), taken by other people with phones
- setting used: training ColorJitter 0.3/0.3/0.3/0.05, dropout 0.3, AdamW weight decay 0.05, label smoothing 0.1. The pretrained network is mobilenet_v3_small (ImageNet), as in Task 2

Results:
variant					seeds	epochs	val		camera-like	old val
baseline				3	15	0.688 ± 0.025	0.312 ± 0.009	0.389 ± 0.057
a0 geometric augmentation		3	15	0.720 ± 0.057	0.446 ± 0.057	0.408 ± 0.046
a geometric + ColorJitter		3	15	0.699 ± 0.009	0.581 ± 0.016	0.414 ± 0.010
b dropout				1	15	0.677		0.339		0.375
c global average pooling		1	15	0.597		0.306		0.408
d AdamW					1	15	0.694		0.387		0.425
e label smoothing			1	15	0.677		0.371		0.458
f best-epoch weights			1	15	0.710		0.290		0.367
g + last years' photos			3	15	0.645 ± 0.043	0.328 ± 0.037	0.753 ± 0.024
h pretrained, only last layer trained	3	15	0.833 ± 0.009	0.801 ± 0.009	0.492 ± 0.008
i pretrained, fully fine-tuned		3	15	0.780 ± 0.025	0.704 ± 0.034	0.511 ± 0.084
j pretrained fine-tuned + augmentation	3	30	0.866 ± 0.037	0.828 ± 0.025	0.619 ± 0.048
+ colour + cosine LR 	
k = j + last years' photos		3	30	0.849 ± 0.019	0.817 ± 0.041	0.903 ± 0.010


What each change brought:
- Geometric augmentation (a0): camera-like improves from 0.31 to 0.45, which is about two standard deviations, so probably real but modest. No clear change on val. Seed 0 alone looked much better (val 0.774), which was luck.
- ColorJitter (a): the clearest gain for the small network. Camera-like rises to 0.58 ± 0.016, well above both the baseline and geometric augmentation alone. It does not help val or old val.
- Last year photos (g): old val jumps from 0.39 to 0.75, far larger than the noise. Val goes down a little (0.645 vs 0.688), but that is within the noise. Camera-like does not change.
- Pretrained network (h, i): the biggest effect of all. Only training the last layer gives camera-like 0.80 instead of 0.31. Fine-tuning the whole network for 15 epochs without augmentation was worse than freezing (0.70 ± 0.03 vs 0.80 ± 0.01).
- Best combination (j): fine-tuning only pays off together with augmentation, the cosine schedule and 30 epochs. Compared with the frozen network it is clearly better on old val (0.62 vs 0.49). On val and camera-like the gain is only about one standard deviation.
- j + last year photos (k): old val goes up to 0.90, and val and camera-like stay the same within the noise.

ColorJitter and the old photos each help on the kind of difference they cover: ColorJitter on changed colours (camera-like), the old photos on other people's leaves and phones (old val). Model k combines both.

Is it larger than the noise?
The validation set is small (62 images) and the baseline's std is 2,5 points on val and 5,7 points on old val. Differences of a few points between single seed runs are therefore noise. Larger than noise: pretrained vs from scratch, ColorJitter on camera-like and the old photos on old val. Within the noise: j vs h on val, j vs k on val and camera-like.

What did not help
Dropout, global average pooling, AdamW, label smoothing and keeping the best epoch showed no difference from the baseline beyond the noise. These were single-seed runs with untuned settings, so I can only say they showed no clear effect here. For the best-epoch weights, the best epoch was the last one, so the result is the same as the baseline.

Caveat
Last year validation photos come from the same people and phones as last year training photos. Once they are used for training (g, k), old val is no longer a test on completely unseen photographers. It still shows that more varied data helps.

The model for the phone
k (pretrained MobileNet, fine-tuned, augmentation with ColorJitter, cosine LR, 30 epochs, plus last year photos). It is the best or tied for best on all three sets and by far the best on other people's photos, which is closest to a new phone camera. I exported seed 1, because its camera-like accuracy (0.823) is closest to the mean of the three seeds (0.817). Its val is 0.871 and its old val is 0.892.