# Advanced Features Implementation Guide

This document provides code snippets and guidance for implementing the advanced features (Helicopter, Yoshi, Koopa, Pikachu, Ice Power) into the main game.

## How to Add These Features to index.html

### Step 1: Add Question Block Types

Find the `QuestionBlock` class constructor and modify it to support different power-up types:

```javascript
class QuestionBlock {
    constructor(x, y, type = 'coin') {
        this.x = x;
        this.y = y;
        this.width = 40;
        this.height = 40;
        this.hit = false;
        this.type = type; // 'coin', 'mushroom', 'helicopter', 'icePower'
        this.bounceOffset = 0;
        this.bouncing = false;
        this.bounceSpeed = 0;
    }

    checkHit(player) {
        if (this.hit) return false;

        if (/* hit detection logic */) {
            this.hit = true;
            this.bouncing = true;
            this.bounceSpeed = -8;

            // Spawn based on type
            if (this.type === 'mushroom') {
                mushrooms.push(new Mushroom(this.x + this.width / 2 - 15, this.y));
            } else if (this.type === 'helicopter') {
                helicopters.push(new Helicopter(this.x, this.y - 50));
            } else if (this.type === 'icePower') {
                icePowers.push(new IcePowerUp(this.x, this.y - 30));
            }

            return true;
        }
        return false;
    }
}
```

### Step 2: Add Ice Power-Up Class

```javascript
class IcePowerUp {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.width = 30;
        this.height = 30;
        this.velocityY = -5;
        this.collected = false;
        this.appearingOffset = -this.height;
        this.appearing = true;
    }

    draw() {
        if (this.collected) return;

        const screenX = this.x - camera.x;
        const screenY = this.y - camera.y + this.appearingOffset;

        // Ice flower
        ctx.fillStyle = '#00ffff';
        ctx.beginPath();
        ctx.arc(screenX + 15, screenY + 15, 12, 0, Math.PI * 2);
        ctx.fill();

        // Petals
        ctx.fillStyle = '#ffffff';
        for (let i = 0; i < 5; i++) {
            const angle = (i / 5) * Math.PI * 2;
            const petalX = screenX + 15 + Math.cos(angle) * 10;
            const petalY = screenY + 15 + Math.sin(angle) * 10;
            ctx.beginPath();
            ctx.arc(petalX, petalY, 5, 0, Math.PI * 2);
            ctx.fill();
        }

        // Center
        ctx.fillStyle = '#0088ff';
        ctx.beginPath();
        ctx.arc(screenX + 15, screenY + 15, 5, 0, Math.PI * 2);
        ctx.fill();
    }

    update() {
        if (this.collected) return;

        if (this.appearing) {
            this.appearingOffset += 1;
            if (this.appearingOffset >= 0) {
                this.appearingOffset = 0;
                this.appearing = false;
            }
            return;
        }

        this.velocityY += GRAVITY;
        this.y += this.velocityY;

        // Platform collision
        platforms.forEach(platform => {
            if (this.checkCollision(platform)) {
                if (this.velocityY > 0 && this.y + this.height - this.velocityY <= platform.y) {
                    this.y = platform.y - this.height;
                    this.velocityY = 0;
                }
            }
        });
    }

    checkCollision(platform) {
        return this.x < platform.x + platform.width &&
               this.x + this.width > platform.x &&
               this.y < platform.y + platform.height &&
               this.y + this.height > platform.y;
    }

    checkCollection(player) {
        if (!this.collected && !this.appearing) {
            const dx = Math.abs((this.x + this.width / 2) - (player.x + player.width / 2));
            const dy = Math.abs((this.y + this.height / 2) - (player.y + player.height / 2));
            if (dx < 30 && dy < 30) {
                this.collected = true;
                return true;
            }
        }
        return false;
    }
}
```

### Step 3: Integrate Helicopter Control

Add to the Player class update method:

```javascript
update() {
    // Helicopter mode
    if (this.helicopter) {
        this.helicopter.update();
        this.x = this.helicopter.x + 10;
        this.y = this.helicopter.y + 15;
        
        // Check dismount
        if ((keys['y'] || keys['Y']) && this.onGround) {
            this.helicopter = null;
            // Remove from helicopters array
            helicopters = helicopters.filter(h => h !== this.helicopter);
        }
        
        camera.x = this.x - canvas.width / 3;
        if (camera.x < 0) camera.x = 0;
        if (camera.x > LEVEL_WIDTH - canvas.width) camera.x = LEVEL_WIDTH - canvas.width;
        
        return;
    }
    
    // Rest of normal update logic...
}
```

### Step 4: Add Mount Detection for Yoshi/Koopa

In the game loop:

```javascript
function gameLoop() {
    // ... existing code ...

    // Update Yoshis
    yoshis.forEach(yoshi => {
        yoshi.update();
        yoshi.draw();
        
        // Check for mounting
        if ((keys['y'] || keys['Y']) && !player.ridingYoshi && !player.ridingKoopa) {
            if (yoshi.checkPlayerCollision(player) && !yoshi.mounted) {
                player.mountYoshi(yoshi);
                keys['y'] = false;
                keys['Y'] = false;
            }
        }
    });

    // Update Koopas
    koopas.forEach(koopa => {
        koopa.update();
        koopa.draw();
        
        // Check for mounting
        if ((keys['y'] || keys['Y']) && !player.ridingYoshi && !player.ridingKoopa) {
            if (koopa.checkPlayerCollision(player) && !koopa.mounted) {
                player.mountKoopa(koopa);
                keys['y'] = false;
                keys['Y'] = false;
            }
        }
    });

    // Update Ice Powers
    icePowers.forEach(power => {
        power.update();
        if (power.checkCollection(player)) {
            player.hasIcePower = true;
            score += 50;
        }
        power.draw();
    });

    // ... rest of game loop ...
}
```

### Step 5: Initialize Arrays

At the top of your script, add:

```javascript
// Arrays for game objects
const platforms = [ /* existing platforms */ ];
const coins = [ /* existing coins */ ];
const questionBlocks = [
    new QuestionBlock(400, 350, 'mushroom'),
    new QuestionBlock(800, 300, 'helicopter'),  // Helicopter spawn
    new QuestionBlock(1200, 350, 'icePower'),   // Ice power spawn
    new QuestionBlock(1600, 400, 'coin'),
    new QuestionBlock(2000, 300, 'helicopter'), // Another helicopter
    // ... more blocks
];

const mushrooms = [];
const fireballs = [];
const iceballs = [];
const goombas = [ /* existing goombas */ ];
const helicopters = [];
const icePowers = [];

// Add Yoshis
const yoshis = [
    new Yoshi(1000, 500),
    new Yoshi(2500, 500),
    new Yoshi(4000, 500)
];

// Add Koopas
const koopas = [
    new Koopa(1500, 500),
    new Koopa(3000, 500),
    new Koopa(4500, 500)
];

// Add Pikachus
const pikachus = [
    new Pikachu(2000, 500),
    new Pikachu(3500, 500)
];
```

### Step 6: Update Info Display

```javascript
// In the HTML section
<div id="info" class="hidden">
    <div>Arrow Keys: Move | Space: Jump | X: Fireball | C: Ice Ball | H: Helicopter | Y: Mount/Dismount</div>
    <div>Hit question blocks for power-ups! Ride Yoshi and Koopa! Stomp or freeze Goombas!</div>
</div>
```

### Step 7: Add Visual Indicators

In the game loop UI section:

```javascript
// Draw power-up indicators
ctx.fillStyle = '#fff';
ctx.font = 'bold 16px Courier New';

if (player.isBig) {
    ctx.fillText('BIG', 20, 110);
}

if (player.hasIcePower) {
    ctx.fillStyle = '#00ffff';
    ctx.fillText('ICE POWER', 20, 130);
    ctx.fillStyle = '#fff';
}

if (player.helicopter) {
    ctx.fillStyle = '#ffff00';
    ctx.fillText('HELICOPTER MODE', 20, 150);
    ctx.fillStyle = '#fff';
}

if (player.ridingYoshi) {
    ctx.fillStyle = '#00ff00';
    ctx.fillText('RIDING YOSHI', 20, 170);
    ctx.fillStyle = '#fff';
}

if (player.ridingKoopa) {
    ctx.fillStyle = '#00ff00';
    ctx.fillText('RIDING KOOPA', 20, 190);
    ctx.fillStyle = '#fff';
}
```

## Testing Checklist

- [ ] Helicopter spawns from question blocks
- [ ] Can mount helicopter with H key
- [ ] Helicopter controls work (arrow keys)
- [ ] Can dismount helicopter with Y key
- [ ] Ice power-up spawns and can be collected
- [ ] Ice power changes player color to cyan
- [ ] Can shoot ice balls with C key
- [ ] Ice balls freeze/defeat enemies
- [ ] Yoshi appears and wanders
- [ ] Can mount Yoshi with Y key near Yoshi
- [ ] Yoshi movement is controlled by player
- [ ] Can dismount Yoshi with Y key
- [ ] Koopa appears and runs fast
- [ ] Can mount Koopa (faster movement)
- [ ] Pikachu appears and walks around
- [ ] All feet touch ground properly

## Common Issues & Solutions

### Issue: Mount key (Y) triggers repeatedly
**Solution**: Reset the key state after mounting:
```javascript
if (keys['y'] || keys['Y']) {
    // Do mount action
    keys['y'] = false;
    keys['Y'] = false;
}
```

### Issue: Helicopter doesn't appear
**Solution**: Make sure helicopters array is initialized and helicopter is added to the array when spawned from question block.

### Issue: Ice balls look the same as fireballs
**Solution**: Use distinct colors (cyan/white for ice vs red/yellow for fire) and different particle effects.

### Issue: Character floats when riding mount
**Solution**: Ensure player position is set relative to mount's top position:
```javascript
this.y = this.ridingYoshi.y - this.height;
```

## Performance Tips

1. **Limit entities**: Don't spawn too many helicopters or mounts at once
2. **Cleanup**: Remove collected power-ups from arrays
3. **Off-screen culling**: Don't update/draw entities far from camera
4. **Projectile limits**: Cap maximum fireballs/iceballs at once

## Good Luck!

You now have all the tools to implement these advanced features. Start with one feature at a time, test thoroughly, then move to the next. The groundwork is already in place in stage1.html - you just need to integrate and test!
