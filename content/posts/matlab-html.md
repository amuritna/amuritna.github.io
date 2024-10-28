---
title: "Presenting MATLAB Figures in a Programmatically Generated HTML Document"
date: 2024-10-28
draft: false
categories: [notes]
tags: [matlab]
description: fprintf, exportgraphics, and taking PDF conversions into consideration.
---

---

As my projects progressed, the number of MATLAB figures to deal with at a time began to go past twenty, and I started developed frustration with MATLAB's default way of arranging and displaying figures. 

Here are some of them:

- Each chart is limited by the size of the figure window, and each figure window is limited by the size of the device screen.
- Therefore, to be able to display more figures, more windows have to launched. I started to feel messy once I started dealing with dozens of figures at a time.
- Individual figures cannot be wider or longer than device screen width or height, respectively.
- Having to manually close or drag around the figure windows every time I re-run the script.
- Most importantly: You can show charts side-by-side in the same window — using subplot or the newer tiledlayout — but control over layout is limited.
    - For instance, tile spacing only has four options: `loose`, `compact`, `tight`, `none` (and `none` does not really *feel* like "no margins", for me).

Since I felt that it was beginning to affect my work in a less ideal way, I began to wonder about the alternatives. What came to mind afterwards was programmatically exporting all the figures created and accessing them using a different application — but we can still go further.

Using MATLAB to "programmatically" export results and generate new files is nothing new, so there was that. The [MATLAB Report Generator](https://www.mathworks.com/help/rptgen/) supports the automatic creation of PDF, Word, PowerPoint, and HTML documents. However, since my primary concern was (lack of) control over layout, I then thought of basic unstyled HTML and the idea of setting up the boilerplate myself. A simple not-particularly-pretty HTML document would be simple to set up, so I became interested in implementing my own generator using native functions.

An additional benefit of using HTML documents was that it meant it would not require an additional add-on, which makes it easier for when I (or people I work) with have to access the code from other devices. Conversion of simple "bare-bones" webpages to PDF are also fast and easy, which would come in handy for presenting my data to my advisor or collaborators.

The following are my notes on this little idea.   

---  

## Exporting graphics with `exportgraphics`
To export figures, [exportgraphics](https://www.mathworks.com/help/matlab/ref/exportgraphics.html) is the way. (You can also use it to automatically produce PDF documents containing only graphics.)

```matlab
f = figure;
plot(filter1_output);
title('Filter 1 Output');

exportgraphics(f, 'img_filter1_output.png');
```

Control over padding, height, and width is available for MATLAB Online. I use the offline version of MATLAB for my current work, however, and adjust image sizes using the figure's [position parameter](https://www.mathworks.com/help/matlab/ref/matlab.ui.figure-properties.html). 

```matlab
f = figure;
f.Position(3:4) = [880 150];
plot(filter1_output);
title('Filter 1 Output');

exportgraphics(f, 'img_filter1_output.png');
```

When iteratively generating and exporting images in a loop, the iterator can be combined with the handle to ensure each generated image has a separate filename:

```matlab
for i = 1:N
    f(i) = figure;
    f(i).Position(3:4) = [880 150];
    plot(filter_outputs(i,:));
    title(['Filter  ', num2str(i), ' Output']);

    exportgraphics(f(1), ['img_filter_', num2str(i), '_output.png']);
```

---  

## Writing new HTML document files using `fopen` + `fprintf`
Basically ...

```matlab
fileID = fopen('hello.html','w');

fprintf(fileID, '<h1>Hello, world!</h1>');

fclose(fileID);
```

---  

## Generating a Boilerplate HTML Document
This is the template I eventually came to use multiple times.

Initially, I thought there was the possibility of me anding having to spend more time doing the "automation" than I would have had I done things manually (or in this case, the default way). Every time, however, it turned out I only need to copy-paste this boilerplate over and over again.

```matlab

%%%%%% DOCUMENT SETUP %%%%%%

fileID = fopen('experiment_3_a_results.html', 'w');
title = sprintf('Experiment 3 Results');
h1 = sprintf('Experiment 3: Results');
subtitle = sprintf('Version A');

document_start = sprintf([ ... 
    '<!DOCTYPE html>' ...
    '<html>' ...
        '<head>' ...
            '<title>', title, '</title>' ...
        '</head>' ...
        '<body>' ...
        '<center>'
]);

document_heading = sprintf([ ...
    '<h1>', h1, '</h1>' ...
    '<p>', subtitle, '</p>' ...
    '<br/>'
    ]);

document_end = sprintf([ ...
        '</body>' ...
        '</center>' ...
    '</html>'
    ]);

fprintf(fileID, document_start);
fprintf(fileID, document_heading);

%%%%%% COMPUTE RESULTS & PLOT %%%%%%

% exportgraphics(plot_handle(i), ['plot_', num2str(i), '.png']);
% fprintf(fileID, ['img src="plot_', num2str(i), '.png" width="880px"/><br/>']);

%%%%%% END DOCUMENT %%%%%%

fprintf(fileID, document_end);
fclose(fileID);
```


A shorter, but less readable, version:
```matlab

%%%%%% DOCUMENT SETUP %%%%%%
fileID = fopen('experiment_3_a_results.html', 'w');

title = sprintf('Experiment 3 Results');
h1 = sprintf('Experiment 3: Results');
subtitle = sprintf('Version A');
fprintf(fileID, ['<!DOCTYPE html><html><head><title>', title, '</title></head><body><center><h1>', h1, '</h1><p>', subtitle, '</p><br/>']);

%%%%%% COMPUTE RESULTS & PLOT %%%%%%

% exportgraphics(plot_handle(i), ['plot_', num2str(i), '.png']);
% fprintf(fileID, ['img src="plot_', num2str(i), '.png" width="880px"/><br/>']);

%%%%%% END DOCUMENT %%%%%%
fprintf(fileID, ['</body></center></html>']);
fclose(fileID);
```

---  

## Configuring Page Breaks (for Converted PDF Document)
One of the properties available in CSS, [break-after](https://developer.mozilla.org/en-US/docs/Web/CSS/break-after), can be used to set page breaks that can be noticed when the document is printed (or, in the usecase I am targeting, when converted to PDF). So, in order to configure page breaks, I insert this:

```html
<div style="break-after: page;"></div>
```

For example, to have each image be rendered on a separate page on the resulting PDF conversion, use:

```matlab
for i = 1:N
    plot_handle(i) = figure;
    plot( '...' );

    exportgraphics(plot_handle(i), ['plot_', num2str(i), '.png']);
    fprintf(fileID, ['img src="plot_', num2str(i), '.png" width="880px"/><br/>']);

    fprintf(fileID, ['<div style="break-after: page;"></div>']);

```

---  

## Afterword
Now that the workflow and results are more to my liking, I am happy dealing with 50+ figures per script nowadays.