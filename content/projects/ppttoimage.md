---
title: "Powershell script to convert PPT slides to images"
date: 2023-11-30T10:00:00+01:00
---

<a href="https://github.com/AkkuRam/ppt_to_image">
  <i class="fab fa-fw fa-github"></i> PPT to Image
</a>

This Powershell script takes in a .pptx file and converts the slides to images

```powershell
function Convert-PPTToImages {
    param (
        [string]$pptFile,
        [string]$outputFolder
    )
    $application = New-Object -ComObject powerpoint.application
    $pres = $application.Presentations.Open($pptFile)

    If(!(test-path -PathType container $outputFolder)){
      New-Item -ItemType Directory -Path $outputFolder
    }
    
    for ($i = 1; $i -le $pres.Slides.Count; $i++) {
        $slide = $pres.Slides.Item($i)
        $outputPath = Join-Path $outputFolder ("Slide" + $i + ".png")
        $slide.Export($outputPath, "PNG")
    }

    $pres.Close()
    $application.Quit()

    [System.Runtime.InteropServices.Marshal]::ReleaseComObject([System.__ComObject]$pres) | out-null
    [System.GC]::Collect()
    [System.GC]::WaitForPendingFinalizers()
}

$pptFile = "$pwd\Example.pptx"
$outputFolder = "$pwd\images"
Convert-PPTToImages -pptFile $pptFile -outputFolder $outputFolder
```