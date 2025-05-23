 // Ateş etme
        function shoot() {
            if (!state.gameRunning) return;
            
            const now = Date.now();
            const weapon = weapons[player.weapon];
            
            if (now - player.lastShot < weapon.delay) return;
            player.lastShot = now;
            
            if (weapon.ammo !== Infinity) {
                if (player.ammo <= 0) return;
                player.ammo--;
            }
            
            if (player.weapon === 'pistol') {
                playSound('shootSound', 0.7);
            } else if (player.weapon === 'shotgun') {
                playSound('shootSound', 0.9);
            } else {
                playSound('shootSound', 0.5);
            }
            
            if (player.weapon === 'shotgun') {
                for (let i = 0; i < weapon.pellets; i++) {
                    createBullet(weapon.spread);
                }
            } else {
                createBullet(weapon.spread);
            }
            
            createMuzzleFlash();
        }

        function createBullet(spread) {
            const angle = Math.atan2(player.direction.y, player.direction.x);
            const spreadAngle = (Math.random() - 0.5) * spread * Math.PI;
            
            state.bullets.push({
                x: player.x,
                y: player.y,
                dx: Math.cos(angle + spreadAngle) * weapons[player.weapon].speed,
                dy: Math.sin(angle + spreadAngle) * weapons[player.weapon].speed,
                radius: weapons[player.weapon].size,
                damage: weapons[player.weapon].damage,
                color: weapons[player.weapon].color
            });
        }

        function createMuzzleFlash() {
            for (let i = 0; i < 5; i++) {
                const angle = Math.atan2(player.direction.y, player.direction.x);
                const spreadAngle = (Math.random() - 0.5) * 0.5 * Math.PI;
                
                state.particles.push({
                    x: player.x + player.direction.x * player.radius,
                    y: player.y + player.direction.y * player.radius,
                    dx: Math.cos(angle + spreadAngle) * (2 + Math.random() * 3),
                    dy: Math.sin(angle + spreadAngle) * (2 + Math.random() * 3),
                    radius: 2 + Math.random() * 4,
                    rotation: Math.random() * Math.PI * 2,
                    rotationSpeed: (Math.random() - 0.5) * 0.2,
                    color: '#f39c12',
                    life: 10 + Math.random() * 10,
                    isExplosion: true
                });
            }
        }

        // Çarpışma kontrolü
        function checkCollisions() {
            if (!state.enemy) return;
            
            for (let i = state.bullets.length - 1; i >= 0; i--) {
                const bullet = state.bullets[i];
                const dx = bullet.x - state.enemy.x;
                const dy = bullet.y - state.enemy.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                if (distance < bullet.radius + state.enemy.radius) {
                    state.enemy.health -= bullet.damage;
                    state.bullets.splice(i, 1);
                    createBloodEffect(state.enemy.x, state.enemy.y, state.enemy.radius);
                    
                    playSound('enemyHitSound', 0.6);
                    
                    if (state.enemy.health <= 0) {
                        state.score += state.enemy.score;
                        state.enemy = null;
                        
                        if (state.score >= 300) {
                            winGame();
                            return;
                        }
                        break;
                    } else {
                        state.enemy.color = '#ffffff';
                        setTimeout(() => {
                            if (state.enemy) state.enemy.color = state.enemy.originalColor;
                        }, 100);
                    }
                }
            }
            
            if (!state.enemy) return;
            
            const dx = player.x - state.enemy.x;
            const dy = player.y - state.enemy.y;
            const distance = Math.sqrt(dx * dx + dy * dy);
            
            if (distance < player.radius + state.enemy.radius) {
                player.health -= 2;
                createBloodEffect(state.enemy.x, state.enemy.y, state.enemy.radius);
                
                playSound('hitSound', 0.5);
                
                if (player.health <= 0) {
                    gameOver();
                }
            }
        }