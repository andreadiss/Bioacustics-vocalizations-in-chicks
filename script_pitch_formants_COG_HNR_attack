########################################################################################
# THIS SCRIPT EXTRACT SOUND FEATURES LOOPING THROUGH EACH TIER AND EACH INTERVAL LABEL.#
# OUTPUT ARE SOUND FEATURES PER DISTRESS CALLS "D" AND PLEASURE CALLS "P".             #
# OUTPUT:                                                                              #
# type: TYPE OF THE CALL                                                               #
# call_n: NUMBER OF THE CALL PER TYPE                                                  #
# start: TIME OF PITCH ONSET                                                           #
# end: TIME OF PITCH END                                                               #
# length: PITCH DURATION                                                               #
# minF0: MINIMUM PITCH                                                                 #
# maxF0: MAXIMUM PITCH                                                                 #
# F0: MEAN PITCH                                                                       #
# F1: MEAN F1                                                                          #
# F2: MEAN F2                                                                          #
# F3: MEAN F3                                                                          #
# COG: CENTRE OF GRAVITY                                                               #
# HNR: HARMONICITY TO NOISE RATIO                                                      #
# ATTACK: TIME TO MAX f0	                                                       #
#                                                                                      #
#!!! CHANGE THE PATH OF THE DIRECTORY !!!					       #                                                   
######################################################################################## 

# SET FIXED PARAMETERS

pitch_floor = 1000
pitch_ceiling = 6000
time_step = 0.001
sample_rate = 44100

#WRITE THE INFO LINE

writeInfoLine: "type", tab$, "call_n",tab$,"start",tab$,"end",tab$,"length",tab$,"maxF0",tab$,"minF0",tab$,"F0",tab$,"Attack_time",tab$,"F1",tab$,"F2",tab$,"F3",tab$,"COG",tab$,"HNR"

# READ FILE FROM DIRECTORY (ONLY THOSE FILES WITH A TEXTGRID)

inputDirectory$= " "

strings=Create Strings as file list... list 'inputDirectory$'/*.TextGrid
numberOfFiles = Get number of strings

for ifile to numberOfFiles
   selectObject: strings
   fileName$ = Get string... ifile
   baseFile$ = fileName$ - ".TextGrid"

# OPEN BOTH TEXTGRID AND WAV FILE WITH THE SAME BASE NAME

   Open long sound file... 'inputDirectory$'/'baseFile$'.wav 
   soundname$ = selected$("LongSound")
   Read from file... 'inputDirectory$'/'baseFile$'.TextGrid
   textgrid$ = selected$("TextGrid")

# COUNT THE DISTRESS AND PLEASURE CALLS (SEE IF LOOPS)

   distress = 0
   pleasure = 0

# EXTRACT NUMBER OF TIERS AND NUMBER OF INTERVALS

  selectObject: "TextGrid 'textgrid$'"
  numberOfTiers = Get number of tiers
	for tierNumber from 1 to numberOfTiers
		selectObject: "TextGrid 'textgrid$'"
		numberOfIntervals = Get number of intervals... tierNumber 
	
# RUN A LOOP FOR EACH INTERVAL TO EXTRACT THE LABEL
	
		for intervalNumber from 1 to numberOfIntervals
			selectObject: "TextGrid 'textgrid$'"
			t1 = Get start point: tierNumber, intervalNumber
       			t2 = Get end point: tierNumber, intervalNumber

# EXTRACT PART OF THE SOUND
			
			select LongSound 'soundname$'
			Extract part... t1 t2 yes

# EXTRACT LABELS
		
			select TextGrid 'textgrid$'
			label$ = Get label of interval... tierNumber intervalNumber

# IF LOOP TO ANALYZE D AND P INDEPENDENTLY
		
			if label$ == "D"
				distress = distress + 1

				@pitchExtraction
			
				@formantsExtraction

				@cogExtraction
			
				@hnrExtraction

				appendInfoLine: "D",tab$,distress,tab$,fixed$(start,3), tab$,fixed$(end,3),tab$, fixed$(length,3),tab$,fixed$(maxF0,3),tab$,fixed$(minF0,3),tab$,fixed$(f0,3),tab$,fixed$(attack,3),tab$,fixed$(f1,3),tab$,fixed$(f2,3),tab$,fixed$(f3,3),tab$,fixed$(cog,3),tab$,fixed$(hnr,3)

# SAME LOOP FOR PLEASURE CALLS

			elsif label$ == "P"
				pleasure = pleasure + 1	
				
				@pitchExtraction
			
				@formantsExtraction

				@cogExtraction
			
				@hnrExtraction

				appendInfoLine: "D",tab$,distress,tab$,fixed$(start,3), tab$,fixed$(end,3),tab$, fixed$(length,3),tab$,fixed$(maxF0,3),tab$,fixed$(minF0,3),tab$,fixed$(f0,3),tab$,fixed$(attack,3),tab$,fixed$(f1,3),tab$,fixed$(f2,3),tab$,fixed$(f3,3),tab$,fixed$(cog,3),tab$,fixed$(hnr,3)

			else
				select Sound 'soundname$'
				Remove

			endif
	
		endfor
	endfor
endfor


# CREATE A PROCEDURE CALLED: pitchExtraction
	
procedure pitchExtraction
	select Sound 'soundname$'
	To Pitch... time_step pitch_floor pitch_ceiling

# EXTRACT START TIME AND END TIME OF THE PITCH (DA CAMBIARE)
		
	select Pitch 'soundname$'
	f = Get frame number from time... t1
	f2 = Get frame number from time... t2
		
	pitchFirst$ = Get value in frame... f Hertz
	while f<f2 and pitchFirst$ = "--undefined-- Hz"
		f = f + 1
		pitchFirst$ = Get value in frame... f Hertz
	endwhile

	start = Get time from frame number... f

	select Pitch 'soundname$'
	f = Get frame number from time... t2
	f1 = Get frame number from time... t1
	pitchLast$ = Get value in frame... f Hertz

	while f>f1 and pitchLast$ = "--undefined-- Hz"
		f = f - 1
		pitchLast$ = Get value in frame... f Hertz
	endwhile

	end = Get time from frame number... f

# GET PITCH DURATION
	length = end - start


# GET THE MAX AND MIN PITCH OF THE TWEET

	select Pitch 'soundname$'
	maxF0 = Get maximum... t1 t2 Hertz Parabolic
	minF0 = Get minimum... t1 t2 Hertz Parabolic
	f0 = Get mean... t1 t2 Hertz

# GET ATTACK DURATION (TIME TO PEAK)
	
	select Sound 'soundname$'
	To Intensity... pitch_floor time_step "yes"

	peakTime = Get time of maximum... t1 t2 Parabolic
	attack=peakTime-start

# REMOVE WINDOWS

	select Pitch 'soundname$'
	Remove
	
	select Intensity 'soundname$'
	Remove

endproc

# CREATE A PROCEDURE CALLED: formantExtraction

procedure formantsExtraction
	select Sound 'soundname$'
	To Formant (burg)... time_step 5 6000 0.025 50 
	f1 = Get mean... 1 t1 t2 Hertz
	f2 = Get mean... 2 t1 t2 Hertz
	f3 = Get mean... 3 t1 t2 Hertz

	select Formant 'soundname$'
	Remove
endproc

# CREATE A PROCEDURE CALLED: cogExtraction
procedure cogExtraction	

	select Sound 'soundname$'
	To Spectrum... "Fast"

	select Spectrum 'soundname$'
	cog= Get centre of gravity... 2

	
	select Spectrum 'soundname$'
	Remove

endproc


# CREATE A PROCEDURE CALLED: hnrExtraction
procedure hnrExtraction	

	select Sound 'soundname$'
	To Harmonicity (cc)... 0.001 1000 0.1 1

	select Harmonicity 'soundname$'
	hnr= Get mean... t1 t2

	select Sound 'soundname$'
	Remove
	
	select Harmonicity 'soundname$'
	Remove

endproc
