+++
title = "Chord Progression Experiment"
date = 2016-01-27
[taxonomies]
tags=["Music","Javascript"]
categories=["Experiment"]
+++
 
Using data on standard pop chord progressions, choose a chord, Major or Minor scale, and step-number range, and get back a chord progression that should (in theory) sound decent.
<!-- more -->
[Try it out here.](https://www.rowlandrose.com/experiments/chord_progression_experiment/index.html)

Inspecting that page and looking at the HTML/JS should make this pretty self-explainatory, but here is the main Javascript:

```javascript
var relative_notes = {
  'I' : 0,
  'i' : 0,
  'bII' : 1,
  'bii' : 1,
  'II' : 2,
  'ii' : 2,
  'bIII' : 3,
  'biii' : 3,
  'III' : 4,
  'iii' : 4,
  'IV' : 5,
  'iv' : 5,
  'bV' : 6,
  'bv' : 6,
  'V' : 7,
  'v' : 7,
  'bVI' : 8,
  'bvi' : 8,
  'VI' : 9,
  'vi' : 9,
  'bVII' : 10,
  'bvii' : 10,
  'VII' : 11,
  'vii' : 11,
}

var note_letters = ['C', 'bD', 'D', 'bE', 'E', 'F', 'bG', 'G', 'bA', 'A', 'bB', 'B'];

var chord_ruleset_html = '';
var chord_json = {};

$.getJSON('chords.json', function(data)
{
  $('#msg').append('Read from chords.json');
  chord_json = data;

  for(var chord_ruleset in data)
  {
    // This avoids going through inherited properties of the object
    if(data.hasOwnProperty(chord_ruleset)) 
    {
      chord_ruleset_html += '<option value="'+chord_ruleset+'">'+chord_ruleset+'</option>';
    }
  }

  $('#chord_ruleset').html(chord_ruleset_html);

});

$('#gen_prog').click(function()
{
  var step_max = parseInt($('#chord_num_max').val());
  var step_min = parseInt($('#chord_num_min').val());
  var step_total = Math.floor( Math.random() * ( ( step_max - step_min ) + 1 ) ) + step_min;
  var current_step = 1;
  var chord_prog = [];
  var danger_chords = [];
  var current_chord = '';

  var chord_data = chord_json[$('#chord_ruleset').val()];

  var key_root = $('#key_root').val();

  // Danger chords: Chords that only lead to root chord. Make sure to avoid these until at the appropriate count away from end.
  // Also, don't make second to last chord the root note. Unless allowing repeat chords.

  // Find danger chords

  var count = 1;
  var root_chord = '';
  for(var chord in chord_data)
  {
    // This avoids going through inherited properties of the object
    if(chord_data.hasOwnProperty(chord)) 
    {
      if(count == 1) {
        root_chord = chord;
        break;
      }
      if(chord_data[chord].length == 1 && chord_data[chord][0] == root_chord) {
        danger_chords.push(chord);
      }
      count++;
    }
  }

  chord_prog.push(root_chord);
  current_step++;
  current_chord = root_chord;

  while(current_step <= step_total)
  {
    var safe_chords = [];
    for(var i = 0; i < chord_data[current_chord].length; i++)
    {
      var possible_chord = chord_data[current_chord][i];
      if(step_total == current_step)
      {
        // Only allow chords with ability to go to root chord
        var safe = false;
        for(var j = 0; j < chord_data[possible_chord].length; j++)
        {
          if(chord_data[possible_chord][j] == root_chord) {
            safe = true;
          }
        }
        if(safe) {
          safe_chords.push(possible_chord);
        }
      }
      else if(step_total - current_step == 1)
      {
        // Don't allow danger chords
        var danger = false;
        for(var j = 0; j < danger_chords.length; j++)
        {
          if(possible_chord == danger_chords[j]) {
            danger = true;
          }
        }
        if(!danger) {
          safe_chords.push(possible_chord);
        }
      }
      else {
        safe_chords.push(possible_chord);
      }
    }

    var chosen_chord = safe_chords[Math.floor(Math.random() * safe_chords.length)];

    chord_prog.push(chosen_chord);
    current_step++;
    current_chord = chosen_chord;
  }

  var output_html = '<p>Step total: ' + step_total + ', Chord progression: ';
  for(var i = 0; i < chord_prog.length; i++)
  {
    if(i > 0) {
      output_html += ' &rarr; ';
    }
    
    var chord_arr = chord_prog[i].split('_');
    var new_chord_num = relative_notes[chord_arr[0]];
    var new_chord = '';
    var suffix_1 = '';
    var suffix_2 = '';

    if(key_root == 'relative') {
      new_chord = chord_arr[0];
    }
    else
    {
      var offset = parseInt(key_root) + new_chord_num;
      if(offset > 11) {
        offset -= 12;
      }
      new_chord = note_letters[offset];
      if(chord_arr[0] == chord_arr[0].toLowerCase()) {
        new_chord = new_chord.toLowerCase();
      }
    }
    // Suffix 1
    if(new_chord.charAt(0) == 'b' && new_chord.charAt(1) != '' ) {
      new_chord = new_chord.substr(1);
      suffix_1 = '&#x266d;';
    }
    // Suffix 2
    if(typeof chord_arr[1] !== 'undefined') {
      var suffix_2 = '_'+chord_arr[1];
      if(suffix_2 == '_d') {
        suffix_2 = '&deg;';
      }
    }
    output_html += new_chord + suffix_1 + suffix_2;
    
  }
  output_html += '</p>';

  $('#chord_output').append(output_html);
});
```

No real reason I do this, but I have the chord information in a separate JSON file:

```json

{
  "Major" : {
    "I" : [
      "ii",
      "iii",
      "IV",
      "V",
      "vi",
      "vii_d"
    ],
    "ii" : [
      "I",
      "V",
      "vii_d"
    ],
    "iii" : [
      "ii",
      "IV",
      "vi"
    ],
    "IV" : [
      "I",
      "ii",
      "V",
      "vii_d"
    ],
    "V" : [
      "I",
      "vi",
      "vii_d"
    ],
    "vi" : [
      "ii",
      "IV"
    ],
    "vii_d" : [
      "V",
      "I"
    ]
  },
  "Minor" : {
    "i" : [
      "ii_d",
      "III",
      "iv",
      "V",
      "VI",
      "vii_d",
      "bVII"
    ],
    "ii_d" : [
      "i",
      "V",
      "vii_d"
    ],
    "III" : [
      "ii_d",
      "iv",
      "VI"
    ],
    "iv" : [
      "i",
      "ii_d",
      "V",
      "vii_d"
    ],
    "V" : [
      "i",
      "VI",
      "vii_d"
    ],
    "VI" : [
      "ii_d",
      "iv"
    ],
    "vii_d" : [
      "i",
      "V",
      "VI"
    ],
    "bVII" : [
      "ii_d",
      "III",
      "iv"
    ]
  },
  "Classical" : {
    "I" : [
      "ii",
      "iii",
      "IV",
      "V",
      "vi",
      "vii"
    ],
    "ii" : [
      "V",
      "vii"
    ],
    "iii" : [
      "IV",
      "vi"
    ],
    "IV" : [
      "ii",
      "V",
      "vii"
    ],
    "V" : [
      "vi"
    ],
    "vi" : [
      "ii",
      "IV",
      "V"
    ],
    "vii" : [
      "I"
    ]
  }
}
```

The main idea is that I define rules about which chords are allowed to progress to which chords, and I randomly choose which of these allowed chords to use.

I based this off of some charts I found on chord progressions. I've lost the actual charts I used but they were something like the top charts on this [StackExchange answer](https://music.stackexchange.com/a/56662).