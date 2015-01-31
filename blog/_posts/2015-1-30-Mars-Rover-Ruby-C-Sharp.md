---
layout: post
date: 2015-01-30 18:40:45
title: Mars Rover problem using Ruby and C#
---

Yet another Ruby and C# adventure. This time it's the __Mars Rover Problem__ (see description [here](http://dallashackclub.com/rover)).

Essentially, the problem is simulate an object (rover) moving on a grid (Mars). Some points on the grid have obstacles. The user can enter runtime commands to move the rover forward (f), backward (b), turn left (l) or turn right (r). Obstacles along the path are reported.

(See full source codes [here](https://github.com/shaurya947/MarsRoverKata))

This time, Ruby code first:

~~~ruby
#Model point as a struct
Point = Struct.new :x, :y

#Grid class's initialize method, places obstacles randomly in 1/10th of the map
def initialize (maxX = 10, minX = -10, maxY = 10, minY = -10)
    @maxX, @minX, @maxY, @minY = maxX, minX, maxY, minY

    @obstacles = Set.new
    numObstacles = ((maxX - minX) * (maxY - minY)) / 10

    (1..numObstacles).each do |n|
        p = Point.new(rand(minX..maxX), rand(minY..maxY))
        if !@obstacles.include?(p)
            @obstacles.add(p)
        end
    end
end

#The Rover class has methods for the movements
class Rover
  def initialize (startingX = 0, startingY = 0, startingDir = :north)
    @currentPos = Point.new(startingX, startingY)
    @currentDir = startingDir
  end

  attr_reader :currentPos, :currentDir

  #method to assign grid to rover
  def assignGrid grid
    @grid = grid

    #if starting point of rover is at an obstacle, move rover over till hit free spot
    if @grid.obstacles.include?(@currentPos)
      print "Grid has obstacle at rover starting position. Landed rover at ("

      loop do
        moveForward
        break if !@grid.obstacles.include?(@currentPos)
      end

      print "#{@currentPos.x}, #{@currentPos.y}) instead\n"
    end
  end

  #method to move rover one step forward
  #display error and return false if obstacle lies ahead
  def moveForward
    case @currentDir
      when :north
        incrementY
      when :south
        decrementY
      when :east
        incrementX
      when :west
        decrementX
    end

    #if new current position has obstacle, move back and report it
    if @grid.obstacles.include?(@currentPos)
      puts "Cannot move forward. Obstacle present ahead at (#{@currentPos.x}, #{@currentPos.y})"
      moveBackward
      return false
    end

    true
  end

  #method to move rover one step backward
  ...

  #method to turn rover to the left
  def turnLeft
    case @currentDir
      when :north
        @currentDir = :west
      when :south
        @currentDir = :east
      when :east
        @currentDir = :north
      when :west
        @currentDir = :south
    end
  end

  #method to turn rover to the right
  ...

  #method to increment rover's x position (with wrap around)
  def incrementX
    @currentPos.x += 1
    if @currentPos.x > @grid.maxX
      @currentPos.x = @grid.minX
    end
  end

  #method to decrement rover's x position (with wrap around)
  ...
  
  #method to display new position
  def displayPosition
    print "New rover position is (#{@currentPos.x}, #{@currentPos.y}) facing "

    case @currentDir
      when :north
        print "North\n"
      when :south
        print "South\n"
      when :east
        print "East\n"
      when :west
        print "West\n"
    end
  end
end
~~~

Now the C# code:

~~~
//Rover class
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MarsRoverKata
{
    //enumerable type to specify direction
    enum Direction { N, S, E, W};

    class Rover
    {
        //properties of Rover
        public Point currentPos { get; private set; }
        public Direction currentDir { get; private set; }

        private Grid grid;     //current grid rover is in

        //constructor that takes initial position and direction
        public Rover(int startingX = 0, int startingY = 0, Direction startingDir = Direction.N)
        {
            currentPos = new Point(startingX, startingY);
            currentDir = startingDir;
        }

        //method to assign grid to rover
        public void assignGrid(Grid grid)
        {
            this.grid = grid;

            if (grid.obstacles.Contains(currentPos))
            {
                Console.Write("Grid has obstacle at rover starting position. Landed rover at ");
                //verify that current position of rover is not an obstacle
                while (grid.obstacles.Contains(currentPos))
                    moveForward();

                Console.Write("(" + currentPos.X + ", " + currentPos.Y + ") instead\n");
            }
        }

        //method to move rover one step forward
        //displays error (and return false) if obstalce lies ahead
        public bool moveForward()
        {
            switch(currentDir)
            {
                case Direction.N: incrementY();
                    break;

                case Direction.S: decrementY();
                    break;

                case Direction.E: incrementX();
                    break;

                case Direction.W: decrementX();
                    break;
            }

            //if new current position has obstacle, move back and report it
            if(grid.obstacles.Contains(currentPos))
            {
                Console.WriteLine("Cannot move forward. Obstacle present ahead at (" + currentPos.X + ", " + currentPos.Y + ")");
                moveBackward();
                return false;
            }

            return true;
        }

        //method to move rover one step backward
        //displays error (and return false) if obstalce lies behind
        ...

        //method to turn rover to the left
        public void turnLeft()
        {
            switch (currentDir)
            {
                case Direction.N: currentDir = Direction.W;
                    break;

                case Direction.S: currentDir = Direction.E;
                    break;

                case Direction.E: currentDir = Direction.N;
                    break;

                case Direction.W: currentDir = Direction.S;
                    break;
            }
        }

        //method to turn rover to the right
        ...

        //method to increment rover's x position (with wrap around)
        private void incrementX()
        {
            int newX = currentPos.X + 1;

            if (newX > grid.maxX)
                newX = grid.minX;

            currentPos = new Point(newX, currentPos.Y);
        }

        //method to decrement rover's x position (with wrap around)
        ...
		
        //method to display new rover position and direction
        public void displayNewPosition()
        {
            Console.Write("New rover position is (" + currentPos.X + ", " + currentPos.Y + ") facing ");

            switch(currentDir)
            {
                case Direction.N: Console.Write("North.\n");
                    break;

                case Direction.S: Console.Write("South.\n");
                    break;

                case Direction.E: Console.Write("East.\n");
                    break;

                case Direction.W: Console.Write("West.\n");
                    break;
            }
        }
    }
}
~~~

To compare the two code segments, I like how I could just use symbols in Ruby and not have to deal with enums (like I had to in C#). It gets a little annoying to prefix the enum type (Direction.) every time. On the other hand, I like the C# sharp model of treating currentPos and currentDir as properties of an object, and _not just_ instance variables.