# jasper-example

# Build Project

git clone --recursive https://github.com/PongoEngine/jasper-example

node Kha/make html5

node Kha/make html5 --server

## Demo
https://pongoengine.github.io/jasper-example/


## Example Code

```haxe

package;

import kha.System;
import jasper.*;

class Main {
    public static function main() : Void
    {
        System.init({title: "jasper-example", width: 800, height: 600}, function() {
            var width = System.windowWidth();
            var height = System.windowHeight();
            var solver = new Solver();

            var window = new Rectangle(0xff444444);
            solver.addConstraint(window.x == 10);
            solver.addConstraint(window.y == 10);
            solver.addConstraint((window.width == width - 20) | Strength.WEAK);
            solver.addConstraint((window.height == height - 20) | Strength.WEAK);

            solver.addConstraint((window.width <= width - 20) | Strength.REQUIRED);
            solver.addConstraint((window.height <= height - 20) | Strength.REQUIRED);
            solver.addConstraint((window.width >= 50) | Strength.REQUIRED);
            solver.addConstraint((window.height >= 50) | Strength.REQUIRED);

            var leftChild = new Rectangle(0xffaaaaaa);
            solver.addConstraint(leftChild.x == window.x + 10);
            solver.addConstraint(leftChild.y == window.y + 10);
            solver.addConstraint(leftChild.width == (window.width/2 - 20));
            solver.addConstraint(leftChild.height == (window.height - 20));

            var rightChild = new Rectangle(0xffdddddd);
            solver.addConstraint(rightChild.x == (window.x + window.width) - (window.width/2 - 10));
            solver.addConstraint(rightChild.y == window.y + 10);
            solver.addConstraint(rightChild.width == (window.width/2 - 20));
            solver.addConstraint(rightChild.height == (window.height - 20));

            var centerChild = new Rectangle(0xffffff00);
            solver.addConstraint(centerChild.x == window.x + window.width/4);
            solver.addConstraint(centerChild.y == window.y + window.height/4);
            solver.addConstraint(centerChild.width == (window.width/2));
            solver.addConstraint(centerChild.height == (window.height/2));

            _rectangles = 
                [ window
                , leftChild
                , rightChild
                , centerChild
                ];
            solver.updateVariables();
            solver.addEditVariable(window.width, Strength.MEDIUM);
            solver.addEditVariable(window.height, Strength.MEDIUM);
            _shouldRender = true;


            System.notifyOnRender(render);

            var _isDown :Bool = false;
            kha.input.Mouse.get().notify(function(b,x,y) {
                _isDown = true;
                solver.suggestValue(window.width, x - 10);
                solver.suggestValue(window.height, y - 10);
                solver.updateVariables();
                _shouldRender = true;
            }, function(b,x,y) {
                _isDown = false;
            }, function(x,y,cX,cY) {
                if(_isDown) {
                    solver.suggestValue(window.width, x - 10);
                    solver.suggestValue(window.height, y - 10);
                    solver.updateVariables();
                    _shouldRender = true;
                }
                
            }, function(c) {
            });
        });
    }

    public static function render(framebuffer: kha.Framebuffer): Void 
    {
        if(_shouldRender) {
            framebuffer.g2.begin(0xffeeeeee);
            for(rectangle in _rectangles) {
                rectangle.render(framebuffer);
            }
            framebuffer.g2.end();
        }
        _shouldRender = false;
    }

    private static var _rectangles :Array<Rectangle>;
    private static var _shouldRender :Bool = true;
}

class Rectangle
{
    public var x :Variable;
    public var y :Variable;

    public var width :Variable;
    public var height :Variable;

    public var color :Int;

    public function new(color :Int) : Void
    {
        this.color = color;
        x = new Variable();
        y = new Variable();
        width = new Variable();
        height = new Variable();
    }

    public function render(framebuffer :kha.Framebuffer) : Void
    {
        framebuffer.g2.color = this.color;
        framebuffer.g2.fillRect(x.m_value, y.m_value, width.m_value, height.m_value);
    }
}
```