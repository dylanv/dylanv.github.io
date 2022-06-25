---
layout: post
title:  "Transfer Learning with ConvNext, HuggingFace, and Ignite"
date:   2022-06-18 16:00:00
categories: ML
---

{% epigraph "The outcome of this exploration is a family of pure ConvNet models dubbed ConvNeXt. Constructed entirely from standard ConvNet modules, ConvNeXts compete favorably with Transformers in terms of accuracy and scalability, achieving 87.8% ImageNet top-1 accuracy and outperforming Swin Transformers on COCO detection and ADE20K segmentation, while maintaining the simplicity and efficiency of standard ConvNets." "Zhuang Liu et al." "A ConvNet for the 2020s" %}

{% newthought "TLDR:" %} I put together a quick-n-dirty example ([Github gist here](https://gist.github.com/dylanv/9b98410ec9cf41f61c4d1ad6baee8822)) if you don't feel like reading.
<!--more-->

{% newthought "Recently, Jeremy Howard" %} [tweeted](https://twitter.com/jeremyphoward/status/1537718111251468288) about some of his experiments with finding which image models are best for [training from scratch](https://www.kaggle.com/code/jhoward/which-image-models-are-best) and for [fine-tuning](https://www.kaggle.com/code/jhoward/the-best-vision-models-for-fine-tuning).
A particular standout was the ConvNeXt model  {% sidenote "one" "Arxiv paper [here](https://arxiv.org/abs/2201.03545). It's pretty readable." %} which is a "modernised" resnet designed to be more similar to transformer models.

This led to a reasonably efficient model that seems to work great on a wide variety of tasks. The [official repo](https://github.com/facebookresearch/ConvNeXt) is pretty helpful and it's built into [timm](https://github.com/rwightman/pytorch-image-models).

{% maincolumn "assets/convnext/comparison.jpg" "Lol (from [Lucas Beyer on twitter](https://twitter.com/giffmana/status/1538617065048788997)" %}

{% newthought "Lately I've been spending" %} way too much time messing around with models from [Hugging Face](https://huggingface.co/).
I've always been a big transfer learning fan, get great results quickly with less compute seems like a no-brainer. 
So I thought this is a good excuse to see how hard it is to get into the inner workings of HuggingFace models 
{% sidenote "two" "It's probably more sensible to use timm but HF also comes with some very useful bells and whistles" %}
and to see if my beloved resnets have been finally replaced. 

{% newthought "I decided" %} to go with the [convnext-tiny-224](https://huggingface.co/facebook/convnext-tiny-224) variant. Who doesn't love a small and performant model?

Grabbing the model is as easy as: {% sidenote "three" "God I love how streamlined some of this is these days." %}
```python
feature_extractor = ConvNextFeatureExtractor.from_pretrained("facebook/convnext-tiny-224")
model = ConvNextForImageClassification.from_pretrained("facebook/convnext-tiny-224")
```
If we go dig around in the guts of it, we can see that replacing the head is as easy as setting `classifier` on the model.
```python
class ConvNextForImageClassification(ConvNextPreTrainedModel):
    def __init__(self, config):
        super().__init__(config)

        self.num_labels = config.num_labels
        self.convnext = ConvNextModel(config)
        # Classifier head
        self.classifier = (
            nn.Linear(config.hidden_sizes[-1], config.num_labels) if config.num_labels > 0 else nn.Identity()
        )
        # Initialize weights and apply final processing
        self.post_init()
```
Cool, so freeze everything, replace the classifier, and you're off to the races? Nope, unfortunatley nothing is ever that easy and Ignite wants your model to return Tensors and the HuggingFace model returns a `ImageClassifierOutputWithNoAttention` object {% sidenote "four" "Or optionally a tuple, but that doesn't actually help at all for the most part." %}
My first thought was to just set a custom train step and grab the logits there, something like:
{% marginnote "one" "Another thought is whether you actually want the model in train mode. Should you update layer norms etc. when transfer learning?" %}
```python
def train_step(trainer, batch):
    model.train()
    optimizer.zero_grad()
    x, y = batch
    x, y = x.to(device), y.to(device)
    # Get a tensor from the return dict
    y_pred = model(x).logits
    loss = loss_fn(y_pred, y)
    loss.backward()
    optimizer.step()
    return loss.item()

trainer = Engine(train_step)
```
This works as long as you don't have any other evaluators or whatever running, which seems fairly unlikely in practice.
{% sidenote "five" "Also we don't want to be writing vanilla training loops like cavemen." %}

{% newthought "In the end" %} the cleanest method was to just wrap the model so that it always returns tensors.
```python
class IgniteConvNext(ConvNextForImageClassification):
    """Wrap a HuggingFace ConvNext classifier so that it can be used with Ignite."""

    def forward(
        self,
        pixel_values: torch.FloatTensor = None,
        labels: Optional[torch.LongTensor] = None,
        output_hidden_states: Optional[bool] = None,
        return_dict: Optional[bool] = None,
    ) -> torch.Tensor:
        return super().forward(pixel_values, labels, output_hidden_states, return_dict=False)[0]
```
This has the other advantage of keeping all the nice saving/loading whatver else you get from the HuggingFace model.

If we're careful setting things up, we can even keep our config matching the model:
```python
# Grab the dataset first so we know our classes. Should have separate folders for each class
dataset = ImageFolder(dataset_folder)

# Grab the pretrained model from Huggingface
model = IgniteConvNext.from_pretrained(pretrained_model_name)

# Swap out the classifier
model.classifier = nn.Linear(model.config.hidden_sizes[-1], len(dataset.classes))
# Update the config
model.config.label2id = dataset.class_to_idx
model.config.id2label = {v: k for k, v in dataset.class_to_idx.items()}
model.config.num_labels = len(dataset.classes)
```

Then freeze the model:
```python
# Freeze all the parameters
for param in model.parameters():
    param.requires_grad = False
# Make sure the classifier part is set to have gradients
model.classifier.requires_grad = True
```
Don't forget to only add the classifier parameters to your optimizer
```python
optimizer = AdamW(model.classifier.parameters(), lr=lr)
```
A really nice part of using HuggingFace is we can also get the feature extractor nice and easy, and then setup of vanilla torch transforms correctly:
```python
# The model comes with an associated feature extractor so grab the transform params from it
feature_extractor = ConvNextFeatureExtractor.from_pretrained(pretrained_model_name)
dataset.transform = T.Compose(
    [
        T.AutoAugment(T.AutoAugmentPolicy.IMAGENET),
        T.PILToTensor(),
        T.ConvertImageDtype(torch.float),
        # Note these are grabbed from the feature_extractor
        T.Normalize(feature_extractor.image_mean, feature_extractor.image_std),
        T.Resize((feature_extractor.size, feature_extractor.size)),
    ]
    )
```
With that you should be off to the races. From my experiments ConvNext does seem to give a performance bump and I like how efficient the smaller versions are. 
{% sidenote "six" "It's also nice getting a working model in 10 minutes on your local machine." %}
Probably going to be using this a lot more in the future.

## Full Example

<script src="https://gist.github.com/dylanv/9b98410ec9cf41f61c4d1ad6baee8822.js"></script>