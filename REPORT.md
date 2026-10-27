# Assignment 2 Report

*Delete this italic guidance as you fill in each section. You'll be asked to
defend any of this without your code in front of you. Write only what you
can actually explain.*

- **Name**: Shamil Mukhamedov
- **Student ID**: 20352
- **Email**: smukhamedov.ieu2022@student.ie.edu
- **Group**: BBADBA 5B

## Business problem, and the cost of being wrong

*The decision this model supports, who or what acts on it, and what happens
when it's wrong in each direction. Be concrete about the asymmetry: which
mistake is worse, roughly by how much, and why. Everything downstream (your
threshold, how much you send to a person, which metric you care about) has to trace
back to this section.*

My project is an image-oriented road-inspection system that detects damaged road surfaces and potholes. Its primary purpose is to help road-maintenance companies identify roads that require inspection and; eventually, potential repair. The does not aim to replace a human inspector - as we discussed in class with the broke bottel example, that is not the point of image-based computer vision models - instead, the model supports prioritization for a road-inspection company.

The two classes that I decided to keep (more on that in the subsequent section) are "Damaged road" and "Potholes". A false negative takes place when a relevant hazard is not detected, potentially delaying inspection and repair. Per contra, a false positive occurs when the system flags a location that does not contain the predicted object, creating unnecessary inspection, resulting in wasted time, money, and labor for the company.

For the initial cost model, a missed hazard is assigned 10 relative cost units, a false positive 2 units, and a human review 1 unit. As such, correct detections incur no error cost. These assumptions are illustrative for the sake of the assignment and will not measured financial losses. The model should therefore prioritize avoiding missed hazards while controlling false alarms for when the work is sent downstream to human reviewers.

My system will be evaluated at various confidence thresholds. In addition, predictions will be routed to automatic acceptance, human review, or no automatic flag according to their confidence. Therefore, the final operating point will be selected using the measured evaluation results and the aforementioned cost assumptions.

## Dataset

*Workspace / project / version on Roboflow Universe, and the link. What one
image represents, what the classes are, and why this dataset actually serves
the problem above rather than merely resembling it.*

The overarching Roboflow dataset had ten classes and more than five datasets, I chose the second one as it initially had three classes and it was fit for a yolov11 model. Not to mention, it has a surplus of images to train the model on. I later found out that the "manhole" class only had four images, which would result in an aggressive class imbalance, so I just removed it entirely using "cp -R PotholeDetection.v2i.yolov11 PotholeDetection.v2i.yolov11_backup" on the bash terminal in my github codepsace. 

Afterwards, I changed the yaml file in the dataset in order to keep the changes consistent across the board to avoid issues in main.py. Later, I also removed the roboflow url portions from the origina yaml file I got while downloading the roboflow dataset since gpt told me that was not necessary to actually do the project properly.

I then created a backup of the original downloaded dataset before modifying the working copy. The working dataset configuration was updated to retain only Damaged road and Potholes, and the four manhole annotations were removed from the training labels. The final model is consequently a two-class object detector.

Each image in the roboflow pothole detection dataset represents a road scene containing zero or more labelled objects. As previously mentioned, the classes represent visible road damage and potholes. The dataset I chose is relevant to the proposed decision because the detected objects can indicate road locations that may need further inspection. However, the dataset alone does not establish the severity of a road defect, its precise location in the road network, or the cost of repairing it, it simply shows whether or not the road has a defect and whether or not the defect is a damaged road or a pothole.

## What the dataset's composition implies

*Instances per class, train/val/test sizes, typical object size relative to the
image. Then the interpretation: which per-class numbers will be too thin to
trust, whether the test split resembles the images you'd really see, and what
you therefore can't conclude from your results.*

The downloaded dataset contains 4,738 images: 4,388 for training, 250 for validation, and 100 for testing.

After excluding the manhole annotations, the retained object counts are as follows: 

Damaged roads
Training objects: 3346
Validation objects: 192
Test objects: 87

Potholes 
Training objects: 1552
Validation objects: 116
Test objects: 55

Total 
Training: 4898
Validtion: 308
Test: 142


The counts above refer to labelled objects and not unique images; therefore, an image may contain more than one object. The training split contains substantially more examples than the validation and test splits, and the Damaged road is more frequent than the Potholes class. On top of that, the test set is relatively small, particularly for per-class evaluation, so performance estimates may be sensitive to individual examples.

The four manhole annotations were confined to the training split. Since no manhole examples appeared in validation or testing, the dataset could not support a reliable evaluation of that class. The limited two-class scope avoids reporting unsupported manhole performance; with that said, the two remaining classes still require inspection for consistent labelling and conceptual overlap.

Object sizes, image conditions, and the representativeness of the test set will be assessed by inspecting the sample images and annotations. Lastly, results should be interpreted as performance on this dataset rather than proof of generalization to every possible road, camera, weather condition, and region - more on that in the limitations section.

## Finetuning and baselines

*Which model and task (detection or classification) and why. A results table on
the same test split: your finetuned model, the pretrained COCO model zero-shot,
and a trivial baseline. What changed, and whether the gap is as large as you
expected.*

My project uses yolov11 for object detection, with the model fine-tuned on two classes. Object detection is appropriate here because the system needs to locate and classify road defects within an image rather than merely classify the entire image. My final fine-tuned model will be compared on the same test split with both the pretrained COCO - Common Objects in COntext - model without fine-tuning and the trivial baseline that makes no detections at all. The predict-nothing baseline cannot detect labelled objects, so it will have zero recall and zero detection average precision. 

Precision is undefined when the model makes no predictions at all. My zero-shot model will be evaluated on the same aforementioned test images, using a documented mapping between its Common Object in COntext - COCO - predictions and the two target classes. Final results will be reported after the evaluations have been made.

## Per-class evaluation and failure modes

*Per-class AP50, or per-class precision/recall and the confusion matrix.
Which class carries the headline number, which one is dragging, and what the
model actually confuses with what. Show where it breaks, not just that it
works.*

The evaluation will report per-class precision, recall, AP50, and the confusion matrix where supported by evaluation procedure. Particular attention will need to be paid in regards to whether the model misses potholes, confuses potholes with a simple road damage, or produces false detections on road surfaces and unrelated objects. Results will be added after training and testing. 

Because the test set contains 87 damaged-road objects and 55 pothole objects, per-class results should be interpreted cautiously. False positives and false negatives will be inspected to identify concrete failure modes.

## Cost matrix and operating point

*Your euro cost for each kind of error and where each figure came from. The
base rate you assumed. The cost-against-threshold sweep, and the operating
point you chose. State the assumption that would most change your answer if it
were wrong.*

My model initially uses relative cost units. More precisely, 10 for a missed hazard, 2 for a false-positive detection, and 1 for a human review. Correct detections have zero additional error cost. As previously mention, these figures are illustrative assumptions rather than measured euro costs. 

The final operating point will be selected by evaluating candidate confidence thresholds on validation data and measuring the resulting error and review costs. The test split will be used for final evaluation rather than repeatedly selecting thresholds. The assumed costs and prevalence of hazards may materially change which operating point is optimal, so the results will be presented as conditional on these assumptions.

## What a person still has to check

*Your two thresholds, and what share of cases they send to a person. How many
that is per day, how long one takes to check, and how many people that needs.
Is that realistic? Then your design priced against the three alternatives:
doing nothing, having a person check everything, and letting the model decide
on its own.*

The dashboard use two confidence thresholds to divide predictions into automatic acceptance, human review, and no automatic flag. The selected thresholds and review share will be reported from the evaluation results. As mentioned above, per-class results should be checked by a person. The review-capacity estimate will state the assumed number of images processed per day and the average time needed to inspect one image. Then, it estimates the daily review workload and the number of reviewers required. The model-assisted policy will be compared with doing nothing, human review of every image, and fully automated model decisions.

## Limitations & next steps

*Real limitations you found, and concretely how you'd address each one with
more time or data. Not generic hedging.*

The test set is small, especially for per-class evaluation. A larger and more diverse held-out dataset would produce more stable performance estimates. The original manhole class was excluded because it had only four training objects and no validation or test objects; consequently, a future version would require a substantially larger and independently evaluated manhole dataset to support that class. For future steps, I would also ensure that this model is applicable across different roads in differen regions under various whether conditions. For instance, where I'm from it tends to snow for nearly 5 months a year, so this model may not work for this since most images in the roboflow dataset I chose have, at most, rainy terrain. 

The labels Damaged road and Potholes have conceptual overlap, which makes sense in the context of the problem for pothole detection. Generally, any type of pothole needs maintenance, but different effort and different laborers for the type of pothole, which is the whole point as mentioned in the business implications. On top of that, sample annotations should be checked for consistency, so a future dataset should define the distinction between general damage and potholes more precisely. The dataset also fails to adequately represent all lighting, camera, and road conditions encountered in real-life deployments.

Lastly, the costs are hypothetical rather than concrete financial estimates. A real deployment needs to estimate the consequences of missed hazards, false alarms, and inspection time using operational data from real financial records from a road-maintenance team in a company.

## Generative AI use disclosure

*Per the syllabus AI Policy: what you used and how, or "no AI content used."*

Generative AI was used to assist with planning the project, debugging and explaining the implementation, and drafting or editing report sections. The final implementation, experimental results, cost assumptions, and conclusions should be checked and verified by the student before submission.