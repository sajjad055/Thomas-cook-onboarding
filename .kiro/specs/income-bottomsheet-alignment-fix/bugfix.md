# Bugfix Requirements Document

## Introduction

On the income screen, clicking the CTA button opens a bottom sheet. The heading and description text elements inside this bottom sheet are not rendering with the correct alignment or spacing. Specifically, both text elements should be left-aligned and separated by 4px of vertical spacing. This fix addresses two visual regressions in the bottom sheet's text layout.

## Bug Analysis

### Current Behavior (Defect)

1.1 WHEN the bottom sheet is opened on the income screen THEN the system displays the heading text with incorrect alignment (not left-aligned)
1.2 WHEN the bottom sheet is opened on the income screen THEN the system displays the description text with incorrect alignment (not left-aligned)
1.3 WHEN the bottom sheet is opened on the income screen THEN the system renders the vertical spacing between the heading and description text with an incorrect gap (not 4px)

### Expected Behavior (Correct)

2.1 WHEN the bottom sheet is opened on the income screen THEN the system SHALL display the heading text with left alignment
2.2 WHEN the bottom sheet is opened on the income screen THEN the system SHALL display the description text with left alignment
2.3 WHEN the bottom sheet is opened on the income screen THEN the system SHALL render exactly 4px of vertical spacing between the heading and description text elements

### Unchanged Behavior (Regression Prevention)

3.1 WHEN the bottom sheet is opened on the income screen THEN the system SHALL CONTINUE TO display and open the bottom sheet correctly upon CTA button click
3.2 WHEN the bottom sheet is opened on the income screen THEN the system SHALL CONTINUE TO render all other bottom sheet content and layout outside of the heading and description text unaffected
3.3 WHEN any other bottom sheet or screen in the application is rendered THEN the system SHALL CONTINUE TO display with its existing alignment and spacing styles unchanged
