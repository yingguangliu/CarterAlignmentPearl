#!/usr/bin/perl

# This program will help align sequences in any FASTA format sequence alignment file.
# You can replace any set of characters with any other set of characters, but
# this means that non-canonical characters can be used. Also, any sequence with the 
# search motif you provide will be affected. There is no 'undo', so save often or
# add your own error-capturing procedures.
#
# This was part of a much larger program, with many different subroutines, hence the
# strangely empty main directory.
#
# No copyright, but this was written by Robert Carter, circa 2020
# I intentionally made this as generic as possible, there are not many of the typically 
# cryptic, "perlish" statements here. Anyone with a grasp of programming should be able 
# to easily adapt this to another language.

$|++;
use strict;
use FileHandle;
use GD;

&Main();

# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *

sub Main(){

   print "\n\n\n\n!*!*!*!*!*!*!*!*!*!*!*!*!*!*!\n\n\n\n";
   print "Sequence Alignment Program\n\n";
   print "   (1) MegAlign\n";
   # Add additional subroutines here
   print "   (x) Exit\n";
   print "\nEnter Selection >";
   my $selection = <STDIN>; chop $selection; print "\n";
   if ($selection eq "1"){&MegAlign;}
   if ($selection eq "99"){
      # insert one-off subroutines here
   }
   elsif ($selection eq "x"){
      exit;
   }
   &Main();
}

# * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *

sub MegAlign(){

   my $skip = 50; my $width = 100; my $start = 0;
   my $input  = "IN.fasta"; # change "IN.fasta" to name of alignment file , e.g. "C:/files/alignment.fas", FASTA format
   my $output = "OUT.fasta"; # change "OUT.fasta" to name of save file
   # make sure you include the entire directory structure in the file name, or add your own way to handle it

   print "   Loading sequences...\n";
   open (IN, $input) || die("Couldn't open file");
   my @data = <IN>; close IN; my $data = join ('', @data); @data = split(/>/, $data); shift @data;
   my $numseqs = @data; print "      $numseqs sequences\n";
   my @names; my @sequences; my $maxlen;
   foreach my $sequence (@data){
      my @sequence = split(/\n/, $sequence); my $name = shift @sequence; my $sequence = join('', @sequence);
      push @names, $name; push @sequences, $sequence; if(length($sequence) > $maxlen){$maxlen = length($sequence);}
   }
   print "\n";
   my $dots = 0;
   for (my $i = $start; $i <= $maxlen; $i = $i + $skip){
      print "\n   X---+----1----+----2----+----3----+----4----+----5----+----6----+----7----+----8----+----9----+----1\n";
      my %hash;
      foreach my $sequence (@sequences){ $hash{substr($sequence, $i, $width)}++; }
      my @firststr = "";
      foreach my $str (sort {$a cmp $b} keys %hash){
         if($dots == 1){
            my $test = @firststr;
            if($test > 1){
               print "   ";
               my @str = split(//, $str);
               for my $j (0..$width - 1){
                  if ($str[$j] eq $firststr[$j]){print ".";}
                  else{print "$str[$j]";}
               }
            }
            else{print "   $str"; @firststr = split (//, $str);}
         }
         else{print "   $str";}
         print " $hash{$str}\n";
      }
      print "   X---+----1----+----2----+----3----+----4----+----5----+----6----+----7----+----8----+----9----+----1\n";
      print "   $i through ", $i + $width, ": (1) Next (2) Last (3) Replace (4) Save (5) Width (6) Skip (7) GoTo (8) Dots (x) quit: ";
      my $selection = <STDIN>; chop $selection;
      if($selection == 2){ $i = $i - 2 * $skip;}
      if($selection == 3){
         print "   Start:"; my $start = <STDIN>; chop $start;
         print "   Old  :"; my $old   = <STDIN>; chop $old  ;
         print "   New  :"; my $new   = <STDIN>; chop $new  ;
         foreach my $sequence (@sequences){
            if(substr($sequence, $i + $start - 1, length($old)) eq $old){
               substr($sequence, $i + $start - 1, length($old)) = "$new";
            }
         }
         $i = $i - $skip;
      }
      if($selection == 4){
         open (OUT, "$output") || die("Couldn't open file");
         for my $i(0..$numseqs - 1){ print OUT ">$names[$i]\n$sequences[$i]\n"; }
         close OUT; $i = $i - $skip;
      }
      if($selection == 5){ print "   New Width:"; my $selection = <STDIN>; chop $selection; $width  = $selection; $i = $i - $skip; }
      if($selection == 6){ print "   New Skip:";  my $selection = <STDIN>; chop $selection; $skip   = $selection; $i = $i - $skip; }
      if($selection == 7){ print "   GoTo Loc:";  my $selection = <STDIN>; chop $selection; $i      = $selection; $i = $i - $skip; }
      if($selection == 8){ $dots = 1 - $dots; $i = $i - $skip;}
      if($selection eq "x"){last;}
   }
}