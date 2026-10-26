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

This project evaluates an image-based road-inspection system that detects damaged road surfaces and potholes. Its purpose is to help road-maintenance teams identify locations that may require inspection or repair. The model supports prioritization rather than replacing a human inspector.

The two retained classes are Damaged road and Potholes. A false negative occurs when a relevant hazard is not detected, potentially delaying inspection or repair. A false positive occurs when the system flags a location that does not contain the predicted object, creating unnecessary inspection work.

For the initial cost model, a missed hazard is assigned 10 relative cost units, a false positive 2 units, and a human review 1 unit. Correct detections incur no additional error cost. These are illustrative assumptions, not measured financial losses. The model should therefore prioritize avoiding missed hazards while controlling false alarms and the workload sent to human reviewers.

The system will be evaluated at different confidence thresholds. Predictions will be routed to automatic acceptance, human review, or no automatic flag according to their confidence. The final operating point will be selected using the measured evaluation results and the stated cost assumptions.

## Dataset

*Workspace / project / version on Roboflow Universe, and the link. What one
image represents, what the classes are, and why this dataset actually serves
the problem above rather than merely resembling it.*

the manhole class only had 4 images, which would result in an INSANE class imbalance, 
so I just removed it entirely using "cp -R PotholeDetection.v2i.yolov11 PotholeDetection.v2i.yolov11_backup"

then changed the yaml file in the dataset in order to keep the changes consistent

also removed the roboflow url portions from the yaml file since gpt told me that was not necessary to actually do the project properly

A backup of the original downloaded dataset was created before modifying the working copy. The working dataset configuration was updated to retain only Damaged road and Potholes, and the four manhole annotations were removed from the training labels. The final model is consequently a two-class object detector.

Each image represents a road scene containing zero or more labelled objects. The retained classes represent visible road damage and potholes. The dataset is relevant to the proposed decision because the detected objects can indicate road locations that may need further inspection. However, the dataset alone does not establish the severity of a defect, its precise location in the road network, or the cost of repairing it.

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


The counts refer to labelled objects, not unique images; an image may contain more than one object. The training split contains substantially more examples than the validation and test splits, and Damaged road is more frequent than Potholes. The test set is relatively small, particularly for per-class evaluation, so performance estimates may be sensitive to individual examples.

The four manhole annotations were confined to the training split. Since no manhole examples appeared in validation or testing, the dataset could not support a reliable evaluation of that class. The two-class scope avoids reporting unsupported manhole performance, although the remaining classes still require inspection for consistent labelling and conceptual overlap.

Object sizes, image conditions, and the representativeness of the test set will be assessed by inspecting sample images and annotations. Results should be interpreted as performance on this dataset rather than proof of generalization to every road, camera, weather condition, or region.

## Finetuning and baselines

*Which model and task (detection or classification) and why. A results table on
the same test split: your finetuned model, the pretrained COCO model zero-shot,
and a trivial baseline. What changed, and whether the gap is as large as you
expected.*

The project uses YOLO11 for object detection, with the model fine-tuned on the two retained classes. Object detection is appropriate because the system needs to locate defects within an image rather than merely classify the entire image.

The fine-tuned model will be compared on the same test split with (1) the pretrained COCO model without fine-tuning and (2) a trivial baseline that makes no detections. Results will be reported after evaluation.


The predict-nothing baseline cannot detect labelled objects, so it has zero recall and zero detection average precision. Precision is undefined when the model makes no predictions. The zero-shot model will be evaluated on the same test images, using a documented mapping between its COCO predictions and the two target classes.

## Per-class evaluation and failure modes

*Per-class AP50, or per-class precision/recall and the confusion matrix.
Which class carries the headline number, which one is dragging, and what the
model actually confuses with what. Show where it breaks, not just that it
works.*

The evaluation will report per-class precision, recall, AP50, and the confusion matrix where supported by the evaluation procedure. Particular attention will be paid to whether the model misses potholes, confuses potholes with general road damage, or produces false detections on road surfaces and unrelated objects.

Results will be added after training and testing. Because the test set contains 87 damaged-road objects and 55 pothole objects, per-class results should be interpreted cautiously. Example false positives and false negatives will be inspected to identify concrete failure modes.

## Cost matrix and operating point

*Your euro cost for each kind of error and where each figure came from. The
base rate you assumed. The cost-against-threshold sweep, and the operating
point you chose. State the assumption that would most change your answer if it
were wrong.*

The initial model uses relative cost units: 10 for a missed hazard, 2 for a false-positive detection, and 1 for a human review. Correct detections have zero additional error cost. These figures are illustrative assumptions rather than measured euro costs.

The final operating point will be selected by evaluating candidate confidence thresholds on validation data and measuring the resulting error and review costs. The test split will be used for final evaluation rather than repeatedly selecting thresholds. The assumed costs and prevalence of hazards may materially change which operating point is optimal, so the results will be presented as conditional on these assumptions.

## What a person still has to check

*Your two thresholds, and what share of cases they send to a person. How many
that is per day, how long one takes to check, and how many people that needs.
Is that realistic? Then your design priced against the three alternatives:
doing nothing, having a person check everything, and letting the model decide
on its own.*

The dashboard will use two confidence thresholds to divide predictions into automatic acceptance, human review, and no automatic flag. The selected thresholds and review share will be reported from the evaluation results.

The review-capacity estimate will state the assumed number of images processed per day and the average time needed to inspect one image. It will then estimate the daily review workload and the number of reviewers required. The model-assisted policy will be compared with doing nothing, human review of every image, and fully automated model decisions.

## Limitations & next steps

*Real limitations you found, and concretely how you'd address each one with
more time or data. Not generic hedging.*

The test set is small, especially for per-class evaluation. A larger and more diverse held-out dataset would produce more stable performance estimates. The original manhole class was excluded because it had only four training objects and no validation or test objects; a future version would require a substantially larger and independently evaluated manhole dataset to support that class.

The labels Damaged road and Potholes may overlap conceptually. Sample annotations should be checked for consistency, and a future dataset should define the distinction between general damage and potholes more precisely. The dataset may also fail to represent all weather, lighting, camera, and road conditions encountered in real deployments.

Finally, the costs are illustrative rather than measured financial estimates. A real deployment should estimate the consequences of missed hazards, false alarms, and inspection time using operational data from a road-maintenance team.

## Generative AI use disclosure

*Per the syllabus AI Policy: what you used and how, or "no AI content used."*

Generative AI was used to assist with planning the project, debugging and explaining the implementation, and drafting or editing report sections. The final implementation, experimental results, cost assumptions, and conclusions should be checked and verified by the student before submission.