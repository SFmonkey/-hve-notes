---
title: 实现 symbol Polyfill
date: 2019-05-07 18:07:28
tags: [blog]
published: true
hideInList: false
feature: 
---
```javascript
		(function() {
        var root = this;

        var generateName = (function () {
            var prefix = 0;

            return function(descriptionStr){
                prefix ++;
                return '@@' + descriptionStr + '_' + prefix;
            }
        })()

        var symbolPoyfill = function (description) {
            if (root instanceof symbolPoyfill) throw new TypeError('symbol is not a constructor');

            var descriptionStr = description == undefined ? undefined : String(description);

            var symbol = Object.create({
                toString: function() {
                    return this.__Description__;
                },
                valueOf: function() {
                    return this;
                }
            });

            Object.defineProperties(symbol, {
                '__Description__' : {
                    value: generateName(descriptionStr),
                    waritable: false,
                    enumerable: false,
                    configurable: false,
                }
            });

            return symbol;
        }

        var forMap = {};

        Object.defineProperties(symbolPoyfill, {
            'for': {
                value: function(key) {
                    forMap[key] ? forMap[key] : forMap[descString] = symbolPoyfill(key)
                },
                waritable: true,
                enumerable: false,
                configurable: true
            },
            'keyFor': {
                value: function(symbol) {
                    for (var key in forMap) {
                        if (forMap[key] === symbol) return key;
                    }
                },
                waritable: true,
                enumerable: false,
                configurable: true
            }
        });

					root.SymbolPolyfill = symbolPoyfill;
			}
		)()
```