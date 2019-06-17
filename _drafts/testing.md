---
title: Testing Jekyll
---

*HELLA WORLD*

* first item
* second item
* third item

{% highlight perl linenos %}
#!/usr/bin/perl -w

use strict;

use LWP;
# use Data::Dumper;

# Create a user agent object
use LWP::UserAgent;
my $ua = LWP::UserAgent->new;
$ua->agent("MemberCounterAndSorterOfAwesomeness/1.0 ");
my @members;
my $base = "wizardofvegas.com";
my $page = 1;
if (defined($ARGV[0]) && $ARGV[0] eq 'DT') {
  $base = "diversitytomorrow.com";
  $page = 0;
}
while ("hella") {
  # Create a request
  my $url = "http://$base/members/$page/";
  my $req = HTTP::Request->new(GET => $url);

  # Pass request to the user agent and get a response back
  my $res = $ua->request($req);

  # Check the outcome of the response
  if ($res->is_success) {
    if ($page != 1 && scalar($res->redirects)) {
      last;
    }
    my $content = $res->content;
    while ($content =~ m{<a (?:class='restricted' )?(?:class='admin' )?href='/member/.+?/' title=".+?">(?<name>.+?)</a></td><td class='members_date'>(?<jdate>.+?)</td><td class='members_threads'>(?<threads>\d+?)</td><td class='members_posts'>(?<posts>\d+?)</td>}g) {
      unless ($+{threads} == 0 && $+{posts} == 0) {
        my %hashCopy = %+;
        push @members, \%hashCopy;
      }
    }
  } else {
    print "Error!\n" . $res->status_line, "\n";
    last;
  }
  $page++;
}

my @sortedMembers = sort { $b->{posts} <=> $a->{posts} } @members;

my $ii = 0;

my $datestamp = qx{TZ='America/Los_Angeles' date -Iminutes};
chomp($datestamp);
print qq{<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8" /><title>$base members by post count</title></head><body>};
print qq{Last updated: $datestamp<br/>};
print qq{<table><tr><th>rank</th><th>member</th><th>posts</th><th>threads</th><th>Posts per day (approximate)</th></tr>\n};

my $now = qx'date +%s';
chomp($now);
foreach my $member(@sortedMembers) {
  my $jepoch = qx{date -d '$member->{jdate}' +\%s};
  chomp($jepoch);
  my $seconds = $now - $jepoch;
  my $days = $seconds / 86400;
  my $postsPerDayRaw = $member->{posts} / $days;
  my $postsPerDay = sprintf("%.2f", $postsPerDayRaw);
  print qq|<tr><td>$ii</td><td>$member->{name}</td><td>$member->{posts}</td><td>$member->{threads}</td><td>$postsPerDay</td></tr>\n|;
  $ii++;
}
print qq{</table></body></html>};
print qq{<!-- MEOW -->};
{% endhighlight %}

{% highlight c %}
/*Lauren Richard
richarla
CS344-400
Homework #4, #2
read a text file and output the unique words to stdout*/

#include <stdio.h>
#include <getopt.h>
#include <sys/utsname.h>
#include <time.h>
#include <sys/stat.h>
#include <string.h>
#include <signal.h>
#include <stdarg.h>
#include <errno.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <ctype.h>

int arrayTrue(int numSort, int *array);
int readWord(int fd, char* buffer);

int main(int argc, char *argv[]){
	char word[50]; //longest english word is 45 characters
	char current [50][51];
	int numSort = 1;
	int i, j;
	int pfd1[50][2];
	int pfd2[50][2];
	FILE* (arrayOfSorts[50]);
	int duplicateCounter;
	char currentWord[50];
	int done[50];
	int pipeEmpty[50];
	
	if (argc == 3 && strcmp(argv[1], "-n") == 0)
	{
		numSort = atoi(argv[2]);
	}


	
	for (i = 0; i < numSort; i++)
	{
		pipe(pfd1[i]);
		pipe(pfd2[i]);
		switch (fork()){
			case -1:
				printf("fork error\n");
			case 0:
        for (j = 0; j < i; j++) {
          //even more unneeded pipes
          close(pfd1[j][0]);
          close(pfd1[j][1]);
          close(pfd2[j][0]);
          close(pfd2[j][1]);
        }
				close(pfd1[i][1]);
				dup2(pfd1[i][0], STDIN_FILENO);
				close(pfd1[i][0]);	
				close(pfd2[i][0]);
				dup2(pfd2[i][1], STDOUT_FILENO);
				close(pfd2[i][1]);			
				execlp("sort", "sort", (char *) NULL);
        break;
			default:
				break;
		}
	}
	j = 0;
	while(scanf("%*[^A-Za-z]"), scanf("%49[A-Za-z]", word) != EOF)
	{
		int sortProcess; 
		for (i = 0; i < strlen(word); i++)
		{
			word[i] = tolower(word[i]);
		}
		//printf("%s\n", word);
		sortProcess = j % numSort;
    write(pfd1[sortProcess][1], word, strlen(word));
		write(pfd1[sortProcess][1], "\n", 1);
		j++;
	}
	//fprintf(stderr, "finished writing to all pipes\n");
	
	for (i = 0; i < numSort; i++)
	{
    close(pfd2[i][1]);
    close(pfd1[i][0]);
		close(pfd1[i][1]);
	}	
	//fprintf(stderr, "finished closing unneeded pipes\n");

	for (i = 0; i < numSort;  i++)
	{
    arrayOfSorts[i] = fdopen(pfd2[i][0], "r");
		if ((fgets(current[i], 51, arrayOfSorts[i])) !=0)
		{
			pipeEmpty[i] = 0;
		}
		else
			pipeEmpty[i] = 1;

	}
	//fprintf(stderr, "created file pointers for all output fds\n");

	while (arrayTrue(numSort, pipeEmpty) == 0)
	{
		int lowestIndex;
		duplicateCounter = 0;
		lowestIndex = 0;
		for (i = 0; i <numSort; i++)
		{
			done[i] = 0;
		}
		for (i = 0; i <numSort; i++)
		{
			if (pipeEmpty[i] == 0)
			{
				lowestIndex = i;
				break;
			}
		}
		
		for (j = lowestIndex; j <numSort; j++)
		{
			if (pipeEmpty[j] == 0)
			{
				if(strcmp(current[lowestIndex], current[j]) > 0)
				{
					lowestIndex = j;
				}
			}
		
		}
		strcpy(currentWord, current[lowestIndex]);
		while(arrayTrue(numSort, done) == 0)
		{
			for (i = 0; i <numSort; i++)
			{

				if (done[i] != 1 && pipeEmpty[i] == 0)
				{
					if(strcmp(currentWord, current[i]) == 0)
					{
						duplicateCounter++;
						if (fgets(current[i], 51, arrayOfSorts[i]) !=0)
						//if (readWord(pfd2[i][0], current[i]) !=0)
						{
							pipeEmpty[i] = 0;
						}
						else
							pipeEmpty[i] = 1;
					}
					else
					{
						done[i] = 1;
					}
				}
				else if (pipeEmpty [i] == 1)
				{
					done[i] = 1;
				}
			}
		}
		printf("%7d %s", duplicateCounter, currentWord);
	}
	//fprintf(stderr, "done, just waiting \n");
	for (i = 0; i < numSort; i++){
		
		wait (NULL);
	}
	//fprintf(stderr, "child processes all terminated \n");
	

	for (i = 0; i < numSort; i++)
	{	
		close(pfd2[i][0]);
	}
	//fprintf(stderr, "all fds closed \n");
	
	return 0;
}

int arrayTrue(int numSort, int *array)
{
	int i;
	for (i = 0; i <numSort; i++)
	{
		if (array[i]== 0)
		{
			return 0;
		}
	}
	return 1;
}

{% endhighlight %}

{% highlight java %}
import java.io.*;

public class CPRange {
    public static void main(String[] args) {
        Runtime r = Runtime.getRuntime();
        try {
            String[] cmd = {"perl", "-e", "use strict; return if !(@ARGV == 3); my $low = $ARGV[0]; my $high = $ARGV[1]; my $destdir = $ARGV[2]; my @files = `ls`; foreach my $file(@files) { chomp $file; my ($n) = $file =~ m/IMG_(\d+).JPG/; if ($n && $n >= $low && $n <= $high) { system \"cp $file $destdir\"; } }", args[0], args[1], args[2] };
            Process p = r.exec(cmd);
            BufferedReader out = new BufferedReader(new InputStreamReader(p.getInputStream()));
            BufferedReader err = new BufferedReader(new InputStreamReader(p.getErrorStream()));
            p.waitFor();
            String outString = out.readLine();
            String errString = err.readLine();
            if (errString != null) {
                System.out.println(errString);
            } else {
                System.out.println(outString);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}