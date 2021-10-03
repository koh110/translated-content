---
title: Utiliser l'API Web Speech
slug: Web/API/Web_Speech_API/Using_the_Web_Speech_API
tags:
  - API
  - API Web Speech
  - Guide
  - Tutoriel
  - Utilisation
  - reconnaissance
  - synthèse
  - vocale
translation_of: Web/API/Web_Speech_API/Using_the_Web_Speech_API
---
L'API Web Speech fournit deux fonctionnalités différentes — la reconnaissance vocale, et la synthèse vocale (aussi appelée "text to speech", ou tts) — qui ouvrent de nouvelles possibiités d'accessibilité, et de mécanismes de contrôle. Cet article apporte une simple introduction à ces deux domaines, accompagnée de démonstrations.

## Reconnaissance vocale

La reconnaissance vocale implique de recevoir de la voix à travers un dispositif de capture du son, tel qu'un microphone, qui est ensuite vérifiée par un service de reconnaissance vocale utilisant une "grammaire" (le vocabulaire que vous voulez faire reconnaître par une application donnée). Quand un mot ou une phrase sont reconnus avec succès, ils sont retournés comme résultat (ou une liste de résultats) sous la forme d'une chaîne de caractères, et d'autres actions peuvent être initiées à la suite de ce résultat.

L'API Web Speech a une interface principale de contrôle  — {{domxref("SpeechRecognition")}} — plus un nombre d'interfaces inter-reliées pour représenter une grammaire, des résultats, etc. Généralement, le système de reconnaissance vocale par défaut disponible sur le dispositif matériel sera utilisé pour la reconnaissance vocale  — la plupart des systèmes d'exploitation modernes ont un système de reonnaissance vocale pour transmettre des commandes vocales. On pense à Dictation sur macOS, Siri sur iOS, Cortana sur Windows 10, Android Speech, etc.

> **Note :** Sur certains navigateurs, comme Chrome, utiliser la reconnaissance vocale sur une page web implique de disposer d'un moteur de reconnaissance basé sur un serveur. Votre flux audio est envoyé à un service web pour traitement, le moteur ne fonctionnera donc pas hors ligne.

### Demo

Pour montrer une simple utilisation de la reconnaissance vocale Web speech, nous avons écrit une demo appelée  [Speech color changer](https://github.com/mdn/web-speech-api/tree/master/speech-color-changer). Quand l'écran est touché ou cliqué, vous pouvez dire un mot clé de couleur HTML et la couleur d'arrière plan de l'application sera modifié par la couleur choisie.

Pour lancer la demo, vous pouvez cloner (ou [directement télécharger](https://github.com/mdn/web-speech-api/archive/master.zip)) le dépôt Github dont elle fait partie, ouvrir le fichier d'index HTML dans un navigateur pour ordinateur de bureau le supportant comme Chrome, ou naviguer vers [l'URL de démonstration live](https://mdn.github.io/web-speech-api/speech-color-changer/), sur un navigateur pour mobile le supportant comme Chrome.

### Support des navigateurs

Le support de la reconnaissance vocale via l'API Web Speech est actuellement limité à Chrome pour ordinateur de bureau et pour mobiles sur Android — Chrome le supporte depuis la version 33 mais avec des interfaces préfixées spécifiques, donc vous avez besoin d'inclure des versions d'interfaces préfixées définies, comme : `webkitSpeechRecognition`.

### HTML and CSS

The HTML and CSS for the app is really trivial. We simply have a title, instructions paragraph, and a div into which we output diagnostic messages.

```html
<h1>Speech color changer</h1>
<p>Tap/click then say a color to change the background color of the app.</p>
<div>
  <p class="output"><em>...diagnostic messages</em></p>
</div>
```

The CSS provides a very simple responsive styling so that it looks ok across devices.

### JavaScript

Let's look at the JavaScript in a bit more detail.

#### Chrome support

As mentioned earlier, Chrome currently supports speech recognition with prefixed properties, therefore at the start of our code we include these lines to feed the right objects to Chrome, and any future implementations that might support the features without a prefix:

```js
var SpeechRecognition = SpeechRecognition || webkitSpeechRecognition
var SpeechGrammarList = SpeechGrammarList || webkitSpeechGrammarList
var SpeechRecognitionEvent = SpeechRecognitionEvent || webkitSpeechRecognitionEvent
```

#### The grammar

The next part of our code defines the grammar we want our app to recognise. The following variable is defined to hold our grammar:

```js
var colors = [ 'aqua' , 'azure' , 'beige', 'bisque', 'black', 'blue', 'brown', 'chocolate', 'coral' ... ];
var grammar = '#JSGF V1.0; grammar colors; public <color> = ' + colors.join(' | ') + ' ;'
```

The grammar format used is [JSpeech Grammar Format](http://www.w3.org/TR/jsgf/) (**JSGF**) — you can find a lot more about it at the previous link to its spec. However, for now let's just run through it quickly:

- The lines are separated by semi-colons, just like in JavaScript.
- The first line — `#JSGF V1.0;` — states the format and version used. This always needs to be included first.
- The second line indicates a type of term that we want to recognise. `public` declares that it is a public rule, the string in angle brackets defines the recognised name for this term (`color`), and the list of items that follow the equals sign are the alternative values that will be recognised and accepted as appropriate values for the term. Note how each is separated by a pipe character.
- You can have as many terms defined as you want on separate lines following the above structure, and include fairly complex grammar definitions. For this basic demo, we are just keeping things simple.

#### Plugging the grammar into our speech recognition

The next thing to do is define a speech recogntion instance to control the recognition for our application. This is done using the {{domxref("SpeechRecognition.SpeechRecognition()","SpeechRecognition()")}} constructor. We also create a new speech grammar list to contain our grammar, using the {{domxref("SpeechGrammarList.SpeechGrammarList()","SpeechGrammarList()")}} constructor.

```js
var recognition = new SpeechRecognition();
var speechRecognitionList = new SpeechGrammarList();
```

We add our `grammar` to the list using the {{domxref("SpeechGrammarList.addFromString()")}} method. This accepts as parameters the string we want to add, plus optionally a weight value that specifies the importance of this grammar in relation of other grammars available in the list (can be from 0 to 1 inclusive.) The added grammar is available in the list as a {{domxref("SpeechGrammar")}} object instance.

```js
speechRecognitionList.addFromString(grammar, 1);
```

We then add the {{domxref("SpeechGrammarList")}} to the speech recognition instance by setting it to the value of the {{domxref("SpeechRecognition.grammars")}} property. We also set a few other properties of the recognition instance before we move on:

- {{domxref("SpeechRecognition.continuous")}}: Controls whether continuous results are captured (`true`), or just a single result each time recognition is started (`false`).
- {{domxref("SpeechRecognition.lang")}}: Sets the language of the recognition. Setting this is good practice, and therefore recommended.
- {{domxref("SpeechRecognition.interimResults")}}: Defines whether the speech recognition system should return interim results, or just final results. Final results are good enough for this simple demo.
- {{domxref("SpeechRecognition.maxAlternatives")}}: Sets the number of alternative potential matches that should be returned per result. This can sometimes be useful, say if a result is not completely clear and you want to display a list if alternatives for the user to choose the correct one from. But it is not needed for this simple demo, so we are just specifying one (which is actually the default anyway.)

```js
recognition.grammars = speechRecognitionList;
recognition.continuous = false;
recognition.lang = 'en-US';
recognition.interimResults = false;
recognition.maxAlternatives = 1;
```

#### Starting the speech recognition

After grabbing references to the output {{htmlelement("div")}} and the HTML element (so we can output diagnostic messages and update the app background color later on), we implement an onclick handler so that when the screen is tapped/clicked, the speech recognition service will start. This is achieved by calling {{domxref("SpeechRecognition.start()")}}. The `forEach()` method is used to output colored indicators showing what colors to try saying.

```js
var diagnostic = document.querySelector('.output');
var bg = document.querySelector('html');
var hints = document.querySelector('.hints');

var colorHTML= '';
colors.forEach(function(v, i, a){
  console.log(v, i);
  colorHTML += '<span style="background-color:' + v + ';"> ' + v + ' </span>';
});
hints.innerHTML = 'Tap/click then say a color to change the background color of the app. Try ' + colorHTML + '.';

document.body.onclick = function() {
  recognition.start();
  console.log('Ready to receive a color command.');
}
```

#### Receiving and handling results

Once the speech recognition is started, there are many event handlers that can be used to retrieve results, and other pieces of surrounding information (see the [`SpeechRecognition` event handlers list](/en-US/docs/Web/API/SpeechRecognition#Event_handlers).) The most common one you'll probably use is {{domxref("SpeechRecognition.onresult")}}, which is fired once a successful result is received:

```js
recognition.onresult = function(event) {
  var color = event.results[0][0].transcript;
  diagnostic.textContent = 'Result received: ' + color + '.';
  bg.style.backgroundColor = color;
  console.log('Confidence: ' + event.results[0][0].confidence);
}
```

The second line here is a bit complex-looking, so let's explain it step by step. The {{domxref("SpeechRecognitionEvent.results")}} property returns a {{domxref("SpeechRecognitionResultList")}} object containing {{domxref("SpeechRecognitionResult")}} objects. It has a getter so it can be accessed like an array — so the first `[0]` returns the `SpeechRecognitionResult` at position 0. Each `SpeechRecognitionResult` object contains {{domxref("SpeechRecognitionAlternative")}} objects that contain individual recognised words. These also have getters so they can be accessed like arrays — the second `[0]` therefore returns the `SpeechRecognitionAlternative` at position 0. We then return its `transcript` property to get a string containing the individual recognised result as a string, set the background color to that color, and report the color recognised as a diagnostic message in the UI.

We also use a {{domxref("SpeechRecognition.onspeechend")}} handler to stop the speech recognition service from running (using {{domxref("SpeechRecognition.stop()")}}) once a single word has been recognised and it has finished being spoken:

```js
recognition.onspeechend = function() {
  recognition.stop();
}
```

#### Handling errors and unrecognised speech

The last two handlers are there to handle cases where speech was recognised that wasn't in the defined grammar, or an error occured. {{domxref("SpeechRecognition.onnomatch")}} seems to be supposed to handle the first case mentioned, although note that at the moment it doesn't seem to fire correctly; it just returns whatever was recognised anyway:

```js
recognition.onnomatch = function(event) {
  diagnostic.textContent = 'I didnt recognise that color.';
}
```

{{domxref("SpeechRecognition.onerror")}} handles cases where there is an actual error with the recognition successfully — the {{domxref("SpeechRecognitionError.error")}} property contains the actual error returned:

```js
recognition.onerror = function(event) {
  diagnostic.textContent = 'Error occurred in recognition: ' + event.error;
}
```

## Speech synthesis

Speech synthesis (aka text-to-speech, or tts) involves receiving synthesising text contained within an app to speech, and playing it out of a device's speaker or audio output connection.

The Web Speech API has a main controller interface for this — {{domxref("SpeechSynthesis")}} — plus a number of closely-related interfaces for representing text to be synthesised (known as utterances), voices to be used for the utterance, etc. Again, most OSes have some kind of speech synthesis system, which will be used by the API for this task as available.

### Demo

To show simple usage of Web speech synthesis, we've provided a demo called [Speak easy synthesis](https://mdn.github.io/web-speech-api/speak-easy-synthesis/). This includes a set of form controls for entering text to be synthesised, and setting the pitch, rate, and voice to use when the text is uttered. After you have entered your text, you can press <kbd>Enter</kbd>/<kbd>Return</kbd> to hear it spoken.

To run the demo, you can clone (or [directly download](https://github.com/mdn/web-speech-api/archive/master.zip)) the Github repo it is part of, open the HTML index file in a supporting desktop browser, or navigate to the [live demo URL](https://mdn.github.io/web-speech-api/speak-easy-synthesis/) in a supporting mobile browser like Chrome, or Firefox OS.

### Browser support

Support for Web Speech API speech synthesis is still getting there across mainstream browsers, and is currently limited to the following:

- Firefox desktop and mobile support it in Gecko 42+ (Windows)/44+, without prefixes, and it can be turned on by flipping the `media.webspeech.synth.enabled` flag to `true` in `about:config`.
- Firefox OS 2.5+ supports it, by default, and without the need for any permissions.
- Chrome for Desktop and Android have supported it since around version 33, without prefixes.

### HTML and CSS

The HTML and CSS are again pretty trivial, simply containing a title, some instructions for use, and a form with some simple controls. The {{htmlelement("select")}} element is initially empty, but is populated with {{htmlelement("option")}}s via JavaScript (see later on.)

```html
<h1>Speech synthesiser</h1>

<p>Enter some text in the input below and press return to hear it. change voices using the dropdown menu.</p>

<form>
  <input type="text" class="txt">
  <div>
    <label for="rate">Rate</label><input type="range" min="0.5" max="2" value="1" step="0.1" id="rate">
    <div class="rate-value">1</div>
    <div class="clearfix"></div>
  </div>
  <div>
    <label for="pitch">Pitch</label><input type="range" min="0" max="2" value="1" step="0.1" id="pitch">
    <div class="pitch-value">1</div>
    <div class="clearfix"></div>
  </div>
  <select>

  </select>
</form>
```

### JavaScript

Let's investigate the JavaScript that powers this app.

#### Setting variables

First of all, we capture references to all the DOM elements involved in the UI, but more interestingly, we capture a reference to {{domxref("Window.speechSynthesis")}}. This is API's entry point — it returns an instance of {{domxref("SpeechSynthesis")}}, the controller interface for web speech synthesis.

```js
var synth = window.speechSynthesis;

var inputForm = document.querySelector('form');
var inputTxt = document.querySelector('.txt');
var voiceSelect = document.querySelector('select');

var pitch = document.querySelector('#pitch');
var pitchValue = document.querySelector('.pitch-value');
var rate = document.querySelector('#rate');
var rateValue = document.querySelector('.rate-value');

var voices = [];
```

#### Populating the select element

To populate the {{htmlelement("select")}} element with the different voice options the device has available, we've written a `populateVoiceList()` function. We first invoke {{domxref("SpeechSynthesis.getVoices()")}}, which returns a list of all the available voices, represented by {{domxref("SpeechSynthesisVoice")}} objects. We then loop through this list — for each voice we create an {{htmlelement("option")}} element, set its text content to display the name of the voice (grabbed from {{domxref("SpeechSynthesisVoice.name")}}), the language of the voice (grabbed from {{domxref("SpeechSynthesisVoice.lang")}}), and `-- DEFAULT` if the voice is the default voice for the synthesis engine (checked by seeing if {{domxref("SpeechSynthesisVoice.default")}} returns `true`.)

We also create `data-` attributes for each option, containing the name and language of the associated voice, so we can grab them easily later on, and then append the options as children of the select.

```js
function populateVoiceList() {
  voices = synth.getVoices();

  for(i = 0; i < voices.length ; i++) {
    var option = document.createElement('option');
    option.textContent = voices[i].name + ' (' + voices[i].lang + ')';

    if(voices[i].default) {
      option.textContent += ' -- DEFAULT';
    }

    option.setAttribute('data-lang', voices[i].lang);
    option.setAttribute('data-name', voices[i].name);
    voiceSelect.appendChild(option);
  }
}
```

When we come to run the function, we do the following. This is because Firefox doesn't support {{domxref("SpeechSynthesis.onvoiceschanged")}}, and will just return a list of voices when {{domxref("SpeechSynthesis.getVoices()")}} is fired. With Chrome however, you have to wait for the event to fire before populating the list, hence the if statement seen below.

```js
populateVoiceList();
if (speechSynthesis.onvoiceschanged !== undefined) {
  speechSynthesis.onvoiceschanged = populateVoiceList;
}
```

#### Speaking the entered text

Next, we create an event handler to start speaking the text entered into the text field. We are using an [onsubmit](/en-US/docs/Web/API/GlobalEventHandlers/onsubmit) handler on the form so that the action happens when <kbd>Enter</kbd>/<kbd>Return</kbd> is pressed. We first create a new {{domxref("SpeechSynthesisUtterance.SpeechSynthesisUtterance()", "SpeechSynthesisUtterance()")}} instance using its constructor — this is passed the text input's value as a parameter.

Next, we need to figure out which voice to use. We use the {{domxref("HTMLSelectElement")}} `selectedOptions` property to return the currently selected {{htmlelement("option")}} element. We then use this element's `data-name` attribute, finding the {{domxref("SpeechSynthesisVoice")}} object whose name matches this attribute's value. We set the matching voice object to be the value of the {{domxref("SpeechSynthesisUtterance.voice")}} property.

Finally, we set the {{domxref("SpeechSynthesisUtterance.pitch")}} and {{domxref("SpeechSynthesisUtterance.rate")}} to the values of the relevant range form elements. Then, with all necessary preparations made, we start the utterance being spoken by invoking {{domxref("SpeechSynthesis.speak()")}}, passing it the {{domxref("SpeechSynthesisUtterance")}} instance as a parameter.

```js
inputForm.onsubmit = function(event) {
  event.preventDefault();

  var utterThis = new SpeechSynthesisUtterance(inputTxt.value);
  var selectedOption = voiceSelect.selectedOptions[0].getAttribute('data-name');
  for(i = 0; i < voices.length ; i++) {
    if(voices[i].name === selectedOption) {
      utterThis.voice = voices[i];
    }
  }
  utterThis.pitch = pitch.value;
  utterThis.rate = rate.value;
  synth.speak(utterThis);
```

In the final part of the handler, we include an {{domxref("SpeechSynthesisUtterance.onpause")}} handler to demonstrate how {{domxref("SpeechSynthesisEvent")}} can be put to good use. When {{domxref("SpeechSynthesis.pause()")}} is invoked, this returns a message reporting the character number and name that the speech was paused at.

```js
   utterThis.onpause = function(event) {
    var char = event.utterance.text.charAt(event.charIndex);
    console.log('Speech paused at character ' + event.charIndex + ' of "' +
    event.utterance.text + '", which is "' + char + '".');
  }
```

Finally, we call [blur()](/en-US/docs/Web/API/HTMLElement/blur) on the text input. This is mainly to hide the keyboard on Firefox OS.

```js
  inputTxt.blur();
}
```

#### Updating the displayed pitch and rate values

The last part of the code simply updates the `pitch`/`rate` values displayed in the UI, each time the slider positions are moved.

```js
pitch.onchange = function() {
  pitchValue.textContent = pitch.value;
}

rate.onchange = function() {
  rateValue.textContent = rate.value;
}
```