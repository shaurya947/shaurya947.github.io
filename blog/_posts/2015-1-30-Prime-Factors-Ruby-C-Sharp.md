---
layout: post
date: 2015-01-30 18:00:00
title: Prime Factors problem using Ruby and C#
---

This is my first time using Ruby and I must say that I am fascinated. The problem, also 
known as __Prime Factors Kata__, was to write a program that would return the prime 
factors of a number in an array. I started out with a program in C# (my _old_ language), 
and then converted that to a Ruby program. In doing so, I tried to take advantage of as many _niceties_ of Ruby as I could. See full source [here](https://github.com/shaurya947/PrimeFactorsKata). First, here is a snippet from the C# program:

~~~
//method that takes in an integer and returns its prime factors as a linkedlist of integers
static int[] generate(int number)
{
    //if number is less than 2, return empty array
    if (number < 2)
        return new int[0];

    //list to store factors along the way
    LinkedList<int> pFactors = new LinkedList<int>();

    //local variables for loop
    int factor = 0;
    bool found = false;

    while(!isPrime(number))
    {
        //find smallest prime number that divides temp
        factor = 2;
        found = false;
		
        while(factor <= number && !found)
        {
            if (isPrime(factor) && number % factor == 0)
                found = true;
			
            else
                factor++;
        }
		

        //add factor to list
        pFactors.AddLast(factor);
        number = number / factor;
    }

    //finally, add temp's data to factors list
    pFactors.AddLast(number);

    return pFactors.ToArray();
	
}
~~~

This code follows the __tree__ approach of prime factorizing a number, wherein the given number is the root of the tree, and leaves are broken down into prime factors until all leaves contain prime numbers. Now, here is the same program in Ruby:

~~~ ruby
class PrimeFactors
    def self.generate(number) #static method to generate prime factors
        if number < 2
            return Array.new
        end
			
        pFactors = Array.new
		
        while !Prime.prime?(number) do
            #find smallest prime number that divides temp
            factor = (2..number).each do |n|
                break n if Prime.prime?(n) && number % n ==0
            end
			
            #add factor to list and reassign variables
            pFactors << factor
            number = number / factor
        end
		
        pFactors << number
    end
end
~~~

Please excuse my bias as I just started with Ruby and am quite liking it. It is probably just a phase. Ruby code feels easier to read. I did not have to create a LinkedList (like I had to in C#) to dynamically append elements to a collection. In Ruby, I could just push numbers to the end of pFactors array. Moreover, Ruby has a built in Prime class that facilitated checking whether or not a number is prime. In C#, I had to write the isPrime method myself.