# CallSum ☎️📖: NLP App for Customer Service Conversation Summarization 
CallSum is a repository for summarization of phone calls. 

<!-- TABLE OF CONTENTS -->
- [CallSum: NLP App for Customer Service Conversation Summarization ☎️](#callsum-nlp-app-for-customer-service-conversation-summarization-️)
  - [About the project ℹ️](#about-the-project-ℹ️)
  - [Libraries and dependencies 📚](#libraries-and-dependencies-)
  - [Generate synthetic data samples](#generate-synthetic-data-samples)
  - [Preprocessing samples 🧰](#preprocessing-samples-)
  - [Fine-tune 🎨](#fine-tune-)
    - [How it works ⚙️](#how-it-works-️)
    - [How to use ⏩](#how-to-use-)
  - [Inference ✍️](#inference-️)
    - [How to use⏩](#how-to-use)
      - [Summarize demo ℹ️](#summarize-demo-ℹ️)
      - [Summarize single files 🗒️](#summarize-single-files-️)
      - [Summarize multiple files 📦](#summarize-multiple-files-)
      - [Summarizing all files in a folder 📁](#summarizing-all-files-in-a-folder-)
    - [Future developments 🏗️](#future-developments-️)
      - [Input token length limitation ⛔](#input-token-length-limitation-)
      - [Changing the model 🔁](#changing-the-model-)
  - [Developers 🔧](#developers-)


## About the project ℹ️

***CallSum*** contains explanations of the development pipeline, including: (i) preprocessing the dataset, (ii) fine-tuning a BART model with that dataset and, (iii) performing inference on the resulting model.


## Libraries and dependencies 📚

 > It is recommended to use Python version 3.10.11 to avoid possible incompatibilities with dependencies and libraries

The first step is to install the required dependencies. Fortunately, the `requirements.txt` file contains all the necessary libraries to run the code without errors. 

```bash
pip install -r requirements.txt
```

## Generate synthetic data samples

To generate synthetic data samples of client conversations, we have used [SynthAI-Datasets 🤖](https://github.com/CICLAB-Comillas/SynthAI-Datasets), which is included as a **submodule** of this repo. To initialize it, execute the following command from a GIT terminal:

```bash
git submodule init
```

Here is an example of a generated synthetic conversation using *SynthAI-Datasets*:
```
"Cliente: ¡Hola! Quería ponerme en contacto con la empresa.

Agente: ¡Hola! ¿En qué le puedo ayudar?

Cliente: Estoy llamando para reportar un problema de cambio de tarifa de luz.

Agente: Está bien, ¿puede proporcionarme su número de teléfono y dirección para seguir adelante?

Cliente: Sí, mi número de teléfono es +34 681382011 y mi dirección es Calle de la cuesta 15.

Agente: Muchas gracias. Por favor, déjeme verificar los detalles de su cuenta. ¿Tiene alguna tarifa específica en mente?

Cliente: Estoy interesado en cambiar a la tarifa diurna.

Agente: Muy bien. Por favor, permita que verifique si esa tarifa se ajusta a sus necesidades.

Agente: ¡He aquí! Hemos verificado sus detalles y hemos encontrado que la tarifa diurna es la mejor para usted. ¿Desea cambiar a esa tarifa?

Cliente: Sí, por favor.

Agente: Está hecho. ¿Hay algo más en lo que le pueda ayudar?

Cliente: No, eso es todo. Muchas gracias por la ayuda.

Agente: De nada. ¡Que tenga un buen día!
``` 

The need to generate synthetic samples arose early in the project to train with data that included both conversations and summaries. In more advanced phases of the project, this module may not be necessary, as the summaries generated by the model itself can be used for this purpose. 

## Preprocessing samples 🧰

Since the conversations provided are in *.transcription* format, a pre-processing is required before the data can be ingested into the model. Typically, one of the most convenient formats for handling large amounts of text is the CSV format, since it allows storing information by fields, which in our case of conversations are 6: *Person, Transcribed line of dialogue, Number of dialogue line, Start time of dialogue line, End time of dialogue time and Quality of transcription*. To automate the process of reading, extraction and conversation, this *Preprocessing* module has been implemented.

The `preprocess.py` file allows to convert the different *.transcription* files into a single CSV file, concatenating the dialogs from each line of the different transcriptions to form a single text string with the entire conversation. 

Before using it, it is good to know the different arguments (flags) accepted by the program:

* `-i`: Input directory path with the .transcription files to preprocess **[Required]** .
* `-o`: Output CSV path. By default it is generated at the input dir.
* `-u`: Flag to indicate whether to generate a single CSV file with all the transcripts (default) or a CSV file for each .transcription.
* `-e`: Input file extension. *.transcription* by default.

The code for executing the preprocessing functionality is:

```bash
python preprocess.py -i <input_dir>
```

Here is a video of the execution of the code for the preprocessing of 128 *.transcription* files, located at *transcriptions* dir:

https://github.com/CICLAB-Comillas/CallSum/assets/59868153/8a3317cd-bf96-4f7c-b22d-87cb3c766d11

After running, the output file is generated as **transcription.csv**.

## Fine-tune 🎨

The fine-tuning process starts with a general model, in this case an already fine-tuned version of the large-sized BART model, specifically, [bart-large-cnn-samsum](https://huggingface.co/philschmid/bart-large-cnn-samsum). This model was first fine-tuned with *CNN Daily Mail*, a large collection of text-summary pairs, and additionally with *Samsun*, a dataset with messenger-like conversations with their summaries.

This section, explains how to fine-tune the complete previous model with a dataset similar to *Samsun* but, in this case, with conversations and summaries in **Spanish**.

### How it works ⚙️

The fine-tuning process will save all the metrics in your *Weights and Biases* account during the training. At the end of the process, the resulting model will also be saved to your **Hugging Face** account. That's why, keys belonging to accounts on both platforms are required to run this code.

The keys need to be stored in a `secrets.json` file in the same directory as the `finetune.py` file using the following format:

```json
{
    "huggingface" : "your_huggingface_access_token",
    "wandb" : "your_wandb_api_key"
}
```

> 🚨 Important reminder: Be careful not to upload your keys. Don't worry, we have taken it into account and this file is included in the .gitignore so that they are not uploaded in any commit.

The remaining part of this section include a brief explanation of the code used to fine-tune in case any modifications are needed on your end, thus, all the comments reference the `finetune.py` file.

The datasets that are going to be loaded, one for training and one for evaluation, need to have the fields `Transcripcion` and `Resumen` so that the code works properly. If you need to change these fieldnames (maybe because you don't have access to changing the fieldnames in the datasets), it is as easy as changing those names in the ***preprocess_function*** function.

> ℹ️ The datasets used for the training are synthetic datasets generated with the OPENAI API exclusively for this task, they do not contain any sensitive information.

In this case, the training dataset contains 10.000 conversations with their summaries in spanish. The evaluation dataset contains roughly 10 conversations since it is used for metric-monitoring purposes. Both can be changed to your own datasets by modifying the ***train_datasets*** and ***eval_datasets*** variables respectively, both can be found below the **LOADING THE DATASET** comment in the code.

The **Training Arguments** can also be changed according to your training and metric-saving preferences, to make the most of these arguments please refer to the original documentation of the `Seq2SeqTrainingArguments` class on **Hugging Face**, which can be found within their [Trainer](https://huggingface.co/docs/transformers/main_classes/trainer) class documentation.

Additionally, the `wandb.init()` method contains information on how to save the data in your **Weights and Biases** acount, such as the project name, run id and run name, amongst others. These parameters can also be changed depending on your preferences.

After the fine-tuning is finished, a wandb alert is displayed to notify the members of the wandb team. Also, the fine-tuned model is saved to your Hugging Face account.


### How to use ⏩
Once you have all the requirements installed, the `secrets.json` file with your keys created and the corresponding changes in the code made (if needed), you can easily run the code with the following command in a console.

```bash
python finetune.py
```

## Inference ✍️

Finally, the last step is inference, which consists of introducing conversations into the model and collecting the results. This process can be tedious, but fortunately, the GUI developed might make things easier. 

### How to use⏩
First, you shall run the inference code:
```bash
python inference.py
```

After a few seconds, it will advise you that the service has been deployed and it's running locally in the following URL: http://127.0.0.1:7860. 


Just **click it** to follow the link and a new tab with the GUI will be opened in a browser. You should see something similar to this: 

![GUI example](https://github.com/CICLAB-Comillas/CallSum/assets/59868153/7683f9d0-d6be-4259-955d-e571fe0e8db0)

At the top of the screen there are 4 tabs, each one has a unique operating mode. We recommend trying the demo mode first as it shows in a more visual way the output of the model.

> 💡 At the bottom of the page there is an accordion element with more details about the modes, click it to display a brief description of them.

#### Summarize demo ℹ️

This one is the simplest. You only have to copy the conversation input as plain text, paste it from the clipboard in the `Transcription` Textbox (left), then click on the `Summarize` button and wait until the output appears in the `Summary` Textbox (right). 

This video shows how to do it:

https://github.com/CICLAB-Comillas/CallSum/assets/59868153/5125a9e7-c967-440e-ba64-156839c46b13

#### Summarize single files 🗒️

This **2nd tab** is used to **summarize a conversation contained in a file**, just load it in the `Input file` box and click on the `Summarize` button. 

In this mode, the conversation is extracted from the input file, then processed and, finally, the generated summary is displayed in the Textbox `Summary`.

Admitted files are:
* **CSV**: The conversation **must belong to the field named `transcription`**.
* **.transcription**: Default format.

Video demo of this 2nd tab:

https://github.com/CICLAB-Comillas/CallSum/assets/59868153/2138e16d-8520-4b3c-b349-10cd6d10c572

#### Summarize multiple files 📦

> 💡 This tab is ideal for processing a large number of conversations in an automated way.

In the third tab, multiple files can be uploaded separately. This mode **allows you to summarize the conversations of each of the files**, generating a CSV file `output.csv` with the results.

This output file contains the following fields:
* `index`: File index (processing order).
* `id`: Identifier, name of the original file without extension.
* `transcription`: Transcription of the conversation (input).
* `summary`: Summary of the transcription generated by the model (output).

**This file `output.csv` can be downloaded by clicking on the [download]() link in the `Output file` box.**

Once the inference has been performed, a message will be displayed in the `Completion` Textbox indicating the percentage of successfully summarized files. In addition, **a list of the files that could not be summarized (if any) will be displayed**.

Here is a video demonstration of how it works:

https://github.com/CICLAB-Comillas/CallSum/assets/59868153/b478791f-6d91-424e-a07d-5ab125091691

#### Summarizing all files in a folder 📁

> 💡 We strongly recommend using the *click-select* method to upload the input files, rather than the drag-and-drop one, as the latter has been found to be buggy (files stuck in *Uploading...* state). 

The last tab is similar to the previous one, only in this case you don't have to worry about manually uploading each file one by one, **just upload the whole folder with all the transcriptions**. This mode is the most optimal for processing large amounts of data.

Demo of how to use this 4th tab:

https://github.com/CICLAB-Comillas/CallSum/assets/59868153/915e5baa-ff08-4006-b16f-361985cfe89e

### Future developments 🏗️

It is likely that some aspects of the `inference.py` code will be replaced in future developments.

#### Input token length limitation ⛔

One of them is the elimination of the `truncate_input` function. This function was developed to avoid the problem of the input token limitation of the current BART model. Since the maximum is 1024, the function makes sure to reduce the input to fit this size, extracting the most relevant parts of the conversation. However, this implies losing a significant part of the information.

If this limitation is solved, the function will no longer be required.

#### Changing the model 🔁

The actual [BARTSum](https://huggingface.co/CICLAB-Comillas/BARTSumpson) model is downloaded using the *pipeline* method from **Hugging Face**. In order to change it, you must edit the following line of code in `inference.py`:

```python
MODEL = "CICLAB-Comillas/BARTSumpson"
```

Just change the `CICLAB-Comillas/BARTSumpson` by your Hugging Face model name.

## Developers 🔧

We would like to thank you for taking the time to read about this project.

If you have any suggestions💡 or doubts❔ **please do not hesitate to contact us**, kind regards from the developers:
  * [Jaime Mohedano](https://github.com/Jatme26)
  * [David Egea](https://github.com/David-Egea)
  * [Ignacio de Rodrigo](https://github.com/nachoDRT)
