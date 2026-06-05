prettycoldworld.github.io[adventure.html](https://github.com/user-attachments/files/28636730/adventure.html)
Seth Krch
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Text Adventure</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: #0d0d0d;
    color: #00ff88;
    font-family: 'Courier New', Courier, monospace;
    font-size: 15px;
    height: 100vh;
    display: flex;
    flex-direction: column;<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Text Adventure</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: #0d0d0d;
    color: #00ff88;
    font-family: 'Courier New', Courier, monospace;
    font-size: 15px;
    height: 100vh;
    display: flex;
    flex-direction: column;
    padding: 16px;
  }
  #output {
    flex: 1;
    overflow-y: auto;
    white-space: pre-wrap;
    line-height: 1.6;
    padding-bottom: 8px;
  }
  #output .line { display: block; }
  #output .dim  { color: #007744; }
  #input-row {
    display: flex;
    align-items: center;
    border-top: 1px solid #005533;
    padding-top: 10px;
    margin-top: 6px;
    gap: 6px;
  }
  #prompt-label { color: #007744; white-space: nowrap; }
  #cmd {
    background: transparent;
    border: none;
    outline: none;
    color: #00ff88;
    font-family: 'Courier New', Courier, monospace;
    font-size: 15px;
    flex: 1;
    caret-color: #00ff88;
  }
  #cmd:disabled { opacity: 0.4; }
</style>
</head>
<body>
<div id="output"></div>
<div id="input-row">
  <span id="prompt-label">&gt;</span>
  <input id="cmd" type="text" autocomplete="off" autofocus spellcheck="false">
</div>

<script>
const out = document.getElementById('output');
const cmd = document.getElementById('cmd');

let state = {};
let _t = 0;

function rnd(min, max) { return Math.floor(Math.random() * (max - min)) + min; }

function p(text, dim, delay) {
  delay = (delay === undefined) ? 120 : delay;
  _t += delay;
  setTimeout(() => {
    const span = document.createElement('span');
    span.className = 'line' + (dim ? ' dim' : '');
    span.textContent = text;
    out.appendChild(span);
    out.scrollTop = out.scrollHeight;
  }, _t);
}

function b(delay) { p('', false, delay || 120); }

function lockInput(extra) {
  cmd.disabled = true;
  setTimeout(() => { cmd.disabled = false; cmd.focus(); }, _t + (extra || 0));
}

function resetTimer() { _t = 0; }

function initGame() {
  state = {
    health: 20, food: 10, gold: 10, attack: 2,
    visits: 0, name: '',
    phase: 'name',
    mob: null, mobHealth: 0, mobAttack: 0,
  };
  resetTimer();
  p('================================');
  p('        TEXT ADVENTURE          ');
  p('================================');
  b();
  p('Enter your name, traveler:');
  lockInput();
}

function showMenu() {
  b();
  p('[ HP:' + state.health + '  Food:' + state.food + '  Gold:' + state.gold + '  ATK:' + state.attack + ' ]', true);
  b();
  p('Select an action:');
  p('1. Move Forward (-1 Food)');
  p('2. Exit Game');
  state.phase = 'menu';
  lockInput();
}

function handleInput(input) {
  input = input.trim();
  resetTimer();
  p('> ' + input, true, 0);

  switch (state.phase) {

    case 'name':
      state.name = input || 'Traveler';
      b();
      p('Welcome, ' + state.name + '. Your journey begins...', false, 400);
      showMenu();
      break;

    case 'menu':
      if (input === '1') {
        state.food--;
        if (state.food < 0) {
          b();
          p('You have starved to death. Game over.', false, 600);
          state.phase = 'dead';
          lockInput();
          break;
        }
        doEncounter();
      } else if (input === '2') {
        b();
        p('Farewell, traveler.', false, 400);
        state.phase = 'dead';
        lockInput();
      } else {
        p('Invalid action.');
        showMenu();
      }
      break;

    case 'chest':
      var loot = rnd(1, 16);
      state.gold += loot;
      p('You heave open the lid...', false, 800);
      b(300);
      p('You found ' + loot + ' gold!', false, 600);
      showMenu();
      break;

    case 'fight':
      doFight(input);
      break;

    case 'tavern_greet':
      doTavernGreet(input);
      break;

    case 'tavern':
      doTavern(input);
      break;

    case 'dead':
      b();
      p('Your adventure has ended. Refresh the page to play again.');
      lockInput();
      break;

    default:
      showMenu();
  }
}

function doEncounter() {
  var roll = rnd(1, 41);

  if (roll < 11) {
    b();
    p('You notice something on the ground...', false, 500);
    b(300);
    p('A chest!', false, 1000);
    b(400);
    p('Press Enter to open it...', false, 300);
    state.phase = 'chest';
    lockInput();

  } else if (roll < 31) {
    b();
    p('You hear a noise...', false, 600);
    b(500);

    var mobType = rnd(1, 4);
    if (mobType === 1) {
      state.mob = 'Skeleton'; state.mobHealth = 10; state.mobAttack = 1;
    } else if (mobType === 2) {
      state.mob = 'Zombie';   state.mobHealth = 5;  state.mobAttack = 5;
    } else {
      state.mob = 'Spider';   state.mobHealth = 2;  state.mobAttack = 6;
    }

    p('A ' + state.mob + ' appears!', false, 1200);
    b(400);
    p('======= ' + state.mob + ' =======', false, 300);
    p('Health: ' + state.mobHealth + '  |  Attack: ' + state.mobAttack, false, 150);
    showFightMenu();

  } else {
    b();
    p('You hear some faint music in the distance...', false, 600);
    b(400);

    if (state.visits === 0) {
      p('Traveler! Great to see you again!!', false, 800);
      b(300);
      p('Did you miss me?', false, 600);
      b();
      p('1) Of course I did.. uh, John.', false, 150);
      p('2) Where the hell am I?',        false, 150);
      p('3) Who the hell are you?',       false, 150);
      state.phase = 'tavern_greet';
      lockInput();
    } else {
      p('Hello, traveler! Good to see you again.', false, 800);
      showTavern();
    }
  }
}

function showFightMenu() {
  b();
  p('[ HP:' + state.health + '  vs  ' + state.mob + ' HP:' + state.mobHealth + ' ]', true);
  b();
  p('Select an action:');
  p('1. Attack');
  p('2. Attempt to Flee');
  p('3. View Loadout');
  p('4. View Stats');
  state.phase = 'fight';
  lockInput();
}

function doFight(input) {
  if (input === '1') {
    b();
    p('Charging attack...', false, 300);

    var damage = state.attack;
    var crit = rnd(1, 6) === 5;

    if (crit) {
      p('Critical hit!', false, 1200);
      damage = state.attack * 2;
    }

    p(damage + ' damage dealt.', false, crit ? 600 : 1200);
    state.mobHealth -= damage;

    if (state.mobHealth <= 0) {
      b(300);
      p(state.mob + ' killed! +5 Gold', false, 600);
      state.gold += 5;
      showMenu();
    } else {
      p('The ' + state.mob + ' has ' + state.mobHealth + ' health remaining.', false, 400);
      b(300);
      p(state.mob + ' prepares an attack...', false, 800);
      state.health -= state.mobAttack;
      p(state.mob + ' dealt ' + state.mobAttack + ' damage. You have ' + state.health + ' health remaining.', false, 1000);
      if (state.health <= 0) {
        b(400);
        p('You have died. Game over.', false, 600);
        state.phase = 'dead';
        lockInput();
      } else {
        showFightMenu();
      }
    }

  } else if (input === '2') {
    b();
    p('You attempt to flee...', false, 600);

    if (rnd(1, 3) === 1) {
      p('You successfully escaped!', false, 1000);
      showMenu();
    } else {
      p('Failed to flee!', false, 1000);
      state.health -= state.mobAttack;
      b(300);
      p(state.mob + ' dealt ' + state.mobAttack + ' damage while you fled. You have ' + state.health + ' health remaining.', false, 600);
      if (state.health <= 0) {
        b(400);
        p('You have died. Game over.', false, 600);
        state.phase = 'dead';
        lockInput();
      } else {
        showFightMenu();
      }
    }

  } else if (input === '3') {
    b();
    p('--- Loadout ---', false, 200);
    p('Attack: ' + state.attack, false, 150);
    showFightMenu();

  } else if (input === '4') {
    b();
    p('--- Stats ---',           false, 200);
    p('Health: ' + state.health, false, 150);
    p('Food:   ' + state.food,   false, 150);
    p('Gold:   ' + state.gold,   false, 150);
    p('Attack: ' + state.attack, false, 150);
    showFightMenu();

  } else {
    p('Invalid action.');
    showFightMenu();
  }
}

function doTavernGreet(input) {
  b();
  if (input === '1') {
    p("That's not my name..", false, 400);
    b(800);
    p('Just messing with you buddy! Welcome to the Tavern!', false, 600);
  } else {
    p('Oh, right. I must have you mistaken for my old buddy ' + state.name + '.', false, 600);
    p('Well, anyways. Welcome to the Tavern!', false, 500);
  }
  state.visits++;
  showTavern();
}

function showTavern() {
  b();
  p('+====Mystic==Tavern====+', false, 400);
  p('Please select an option:', false, 200);
  p('1. Leave Tavern',          false, 150);
  state.phase = 'tavern';
  lockInput();
}

function doTavern(input) {
  if (input === '1') {
    b();
    p('You head back out into the wilderness.', false, 400);
    showMenu();
  } else {
    p('Invalid option.');
    showTavern();
  }
}

cmd.addEventListener('keydown', function(e) {
  if (e.key === 'Enter' && !cmd.disabled) {
    var val = cmd.value;
    cmd.value = '';
    handleInput(val);
  }
});

initGame();
</script>
</body>
</html>

    padding: 16px;
  }
  #output {
    flex: 1;
    overflow-y: auto;
    white-space: pre-wrap;
    line-height: 1.6;
    padding-bottom: 8px;
  }
  #output .line { display: block; }
  #output .dim  { color: #007744; }
  #input-row {
    display: flex;
    align-items: center;
    border-top: 1px solid #005533;
    padding-top: 10px;
    margin-top: 6px;
    gap: 6px;
  }
  #prompt-label { color: #007744; white-space: nowrap; }
  #cmd {
    background: transparent;
    border: none;
    outline: none;
    color: #00ff88;
    font-family: 'Courier New', Courier, monospace;
    font-size: 15px;
    flex: 1;
    caret-color: #00ff88;
  }
</style>
</head>
<body>
<div id="output"></div>
<div id="input-row">
  <span id="prompt-label">&gt;</span>
  <input id="cmd" type="text" autocomplete="off" autofocus spellcheck="false">
</div>

<script>
const out = document.getElementById('output');
const cmd = document.getElementById('cmd');

let state = {};

function rnd(min, max) { return Math.floor(Math.random() * (max - min)) + min; }

function print(text, dim) {
  const span = document.createElement('span');
  span.className = 'line' + (dim ? ' dim' : '');
  span.textContent = text;
  out.appendChild(span);
  out.scrollTop = out.scrollHeight;
}

function blank() { print(''); }

function initGame() {
  state = {
    health: 20,
    food: 10,
    gold: 10,
    attack: 2,
    visits: 0,
    name: '',
    phase: 'name',
    mob: null,
    mobHealth: 0,
    mobAttack: 0,
  };
  print('================================');
  print('        TEXT ADVENTURE          ');
  print('================================');
  blank();
  print('Enter your name, traveler:');
}

function showMenu() {
  blank();
  print(`[ HP:${state.health}  Food:${state.food}  Gold:${state.gold}  ATK:${state.attack} ]`, true);
  blank();
  print('Select an action:');
  print('1. Move Forward (-1 Food)');
  print('2. Exit Game');
  state.phase = 'menu';
}

function handleInput(input) {
  input = input.trim();
  print('> ' + input, true);

  switch (state.phase) {

    case 'name':
      state.name = input || 'Traveler';
      blank();
      print(`Welcome, ${state.name}. Your journey begins...`);
      showMenu();
      break;

    case 'menu':
      if (input === '1') {
        state.food--;
        if (state.food < 0) {
          blank();
          print('You have starved to death. Game over.');
          state.phase = 'dead';
          break;
        }
        doEncounter();
      } else if (input === '2') {
        blank();
        print('Farewell, traveler.');
        state.phase = 'dead';
      } else {
        print('Invalid action.');
        showMenu();
      }
      break;

    case 'chest':
      const loot = rnd(1, 16);
      state.gold += loot;
      blank();
      print(`You open the chest and find ${loot} gold!`);
      setTimeout(() => showMenu(), 0);
      break;

    case 'fight':
      doFight(input);
      break;

    case 'tavern_greet':
      doTavernGreet(input);
      break;

    case 'tavern':
      doTavern(input);
      break;

    case 'dead':
      blank();
      print('Your adventure has ended. Refresh the page to play again.');
      break;

    default:
      showMenu();
  }
}

function doEncounter() {
  const roll = rnd(1, 41);

  if (roll < 11) {
    blank();
    print('A chest appears before you.');
    blank();
    print('Press Enter to open it...');
    state.phase = 'chest';

  } else if (roll < 31) {
    blank();
    print('You hear a noise...');
    blank();
    const mobType = rnd(1, 4);
    if (mobType === 1) {
      state.mob = 'Skeleton'; state.mobHealth = 10; state.mobAttack = 1;
    } else if (mobType === 2) {
      state.mob = 'Zombie';   state.mobHealth = 5;  state.mobAttack = 5;
    } else {
      state.mob = 'Spider';   state.mobHealth = 2;  state.mobAttack = 6;
    }
    print(`A ${state.mob} appears!`);
    blank();
    print(`======= ${state.mob} =======`);
    print(`Health: ${state.mobHealth}  |  Attack: ${state.mobAttack}`);
    blank();
    showFightMenu();

  } else {
    blank();
    print('You hear some faint music in the distance...');
    blank();
    if (state.visits === 0) {
      print('Traveler! Great to see you again!!');
      blank();
      print('Did you miss me?');
      blank();
      print('1) Of course I did.. uh, John.');
      print('2) Where the hell am I?');
      print('3) Who the hell are you?');
      state.phase = 'tavern_greet';
    } else {
      print('Hello, traveler! Good to see you again.');
      blank();
      showTavern();
    }
  }
}

function showFightMenu() {
  print(`[ HP:${state.health}  vs  ${state.mob} HP:${state.mobHealth} ]`, true);
  blank();
  print('Select an action:');
  print('1. Attack');
  print('2. Attempt to Flee');
  print('3. View Loadout');
  print('4. View Stats');
  state.phase = 'fight';
}

function doFight(input) {
  if (input === '1') {
    blank();
    print('Charging attack...');
    let damage = state.attack;
    if (rnd(1, 6) === 5) {
      print('Critical hit!');
      damage = state.attack * 2;
    }
    print(`${damage} damage dealt.`);
    state.mobHealth -= damage;

    if (state.mobHealth <= 0) {
      blank();
      print(`${state.mob} killed! +5 Gold`);
      state.gold += 5;
      showMenu();
    } else {
      print(`The ${state.mob} has ${state.mobHealth} health remaining.`);
      blank();
      print(`${state.mob} prepares an attack...`);
      state.health -= state.mobAttack;
      print(`${state.mob} dealt ${state.mobAttack} damage. You have ${state.health} health remaining.`);
      if (state.health <= 0) {
        blank();
        print('You have died. Game over.');
        state.phase = 'dead';
      } else {
        blank();
        showFightMenu();
      }
    }

  } else if (input === '2') {
    if (rnd(1, 3) === 1) {
      blank();
      print('You successfully fled!');
      showMenu();
    } else {
      blank();
      print('Failed to flee!');
      state.health -= state.mobAttack;
      print(`${state.mob} dealt ${state.mobAttack} damage while you fled. You have ${state.health} health remaining.`);
      if (state.health <= 0) {
        blank();
        print('You have died. Game over.');
        state.phase = 'dead';
      } else {
        blank();
        showFightMenu();
      }
    }

  } else if (input === '3') {
    blank();
    print('--- Loadout ---');
    print(`Attack: ${state.attack}`);
    blank();
    showFightMenu();

  } else if (input === '4') {
    blank();
    print('--- Stats ---');
    print(`Health: ${state.health}`);
    print(`Food:   ${state.food}`);
    print(`Gold:   ${state.gold}`);
    print(`Attack: ${state.attack}`);
    blank();
    showFightMenu();

  } else {
    print('Invalid action.');
    blank();
    showFightMenu();
  }
}

function doTavernGreet(input) {
  blank();
  if (input === '1') {
    print("That's not my name..");
    blank();
    print('Just messing with you buddy! Welcome to the Tavern!');
  } else {
    print(`Oh, right. I must have you mistaken for my old buddy ${state.name}.`);
    print('Well, anyways. Welcome to the Tavern!');
  }
  state.visits++;
  blank();
  showTavern();
}

function showTavern() {
  print('+====Mystic==Tavern====+');
  print('Please select an option:');
  print('1. Leave Tavern');
  state.phase = 'tavern';
}

function doTavern(input) {
  if (input === '1') {
    blank();
    print('You head back out into the wilderness.');
    showMenu();
  } else {
    print('Invalid option.');
    blank();
    showTavern();
  }
}

cmd.addEventListener('keydown', function(e) {
  if (e.key === 'Enter') {
    const val = cmd.value;
    cmd.value = '';
    handleInput(val);
  }
});

initGame();
</script>
</body>
</html>
