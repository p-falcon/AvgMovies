# AvgMovies
%Pedro Falcon
%01/12/2017
%MATLAB Script that will be used to average frames in a movie. 
%% %% Identifying file and importing movie

%Getting image and info 
[FileName,PathName] = uigetfile('*.tif','Select image to be processed/analyzed');
SplitFileName = strsplit(FileName,'.');
SplitFileName_1 = char(SplitFileName{1});
InfoImage = imfinfo(FileName);

%Getting image size and # of frames from tiff stack
nImage=InfoImage(1).Height;
mImage=InfoImage(1).Width;
NumberImages=length(InfoImage);

%Preallocating 3D matrix for tiff stack
ImportedStack=zeros(nImage,mImage,NumberImages,'uint8');

%nImage = vertical length of Matrix
%mImage = horizontal length of Matrix
 
%Populating 3D matrix from tiff stack 
TifLink = Tiff(FileName, 'r');
for i = 1:NumberImages
   TifLink.setDirectory(i);
   ImportedStack(:,:,i )= TifLink.read();
end
TifLink.close();

%% Loading GCaMP timeseries and timepoints from Fiji preprocessing
TimePointDataPath = strcat(PathName,'/TimePointData/');
cd(TimePointDataPath);
SplitTitle = strsplit(SplitFileName_1,'_');
    %Because TimePointFileName doesn't have the appended _A
    %Timepoints in movie don't depend on analysis, and so they are connected to
    %the raw movie.  .dat file could be altered to include the analysis in the
    %timepoint label if necessary.  
TimePointFileName = strcat(char(SplitTitle{1}),'_',char(SplitTitle{2}),'_',char(SplitTitle{3}),'_TimePoints.dat');
TimePoints=csvread(TimePointFileName);

TimePoints(2) = 231; %Beginning of ovulation
TimePoints(3) = 233; %Ending of ovulation
lengthOfTime = TimePoints(3) - TimePoints(2) + 1;
%% 
prompt1 = '\nIndicate the desired amount of frames that will be averaged: ';
windowLength = input(prompt1);

newMovieLength = lengthOfTime/windowLength; %Number of frames in new movie
remainder = rem(lengthOfTime,windowLength); %Calculates remainder to check if frames of new movie should be rounded up

if(remainder ~= 0) 
    newMovieLength = ceil(newMovieLength); %If remainder is 0, the # of frames in new movie will be rounded up
end
    

newMov = zeros(nImage,mImage,newMovieLength); %Matrix that will hold new mean values
frame = TimePoints(2); %Frame is initialized when ovulation begins.

n=1; %Counter in for loop. Keeps track of frames in new movie.
j = 1; %Will determine which frames to be considered for calculation of mean values. 

for n=1:newMovieLength
    newMov(:,:,n) = ImportedStack(:,:,frame);
    for j=1:windowLength-1
        if(frame ~= TimePoints(3)) %Checks to see if last frame is being read
            newMov(:,:,n) = (newMov(:,:,n)+ImportedStack(:,:,frame+1));
            frame = frame + 1;
        else
            newMov(:,:,n) = (newMov(:,:,n) + ImportedStack(:,:,frame));
            fprintf('f');
        end
    end
    frame = frame + 1;
    newMov(:,:,n) = newMov(:,:,n)/windowLength; %Gets average value
end
