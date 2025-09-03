package com.bnpparibas.dpw.helpers;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

import com.bnpparibas.dpw.entity.dpw.SpecialInstructionsEntity;
import com.bnpparibas.dpw.repository.SpecialInstructionsRepository;
import com.lowagie.text.Element;
import com.lowagie.text.pdf.ColumnText;
import com.lowagie.text.pdf.PdfContentByte;
import com.lowagie.text.pdf.PdfPageEventHelper;
import com.lowagie.text.DocumentException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import com.bnpparibas.dpw.controller.model.RequestEvent;
import com.lowagie.text.Chunk;
import com.lowagie.text.Document;
import com.lowagie.text.Font;
import com.lowagie.text.FontFactory;
import com.lowagie.text.PageSize;
import com.lowagie.text.Paragraph;
import com.lowagie.text.pdf.PdfWriter;

@Component
public class DpwEventPdfExporter {

    private static final String MFR = "MFR: ";
    private static final String UMFR = "UMFR: ";
    private static final String EVENTID = "Conserto Event ID: ";
    private static final String CUSTOMER_NAME = "Customer Name: ";
    private static final String CUSTOMER_ID = "Customer ID: ";
    private static final String PRODUCT_GROUP = "Product Group: ";
    private static final String PRODUCT = "Product: ";
    private static final String EVENT = "Event: ";
    private static final String EVENT_PRIORITY = "Event Priority: ";
    private static final String EVENT_INITIATION = "Event Initiation: ";
    private static final String SPECIAL_INSTRUCTION = "Special Instruction: ";

    @Value("${event.coversheet.spacing.betweenproperties}")
    private String betweenProperties;

    @Value("${event.coversheet.spacing.afterheader}")
    private String afterHeader;

    @Value("${event.coversheet.spacing.beforefooter}")
    private String beforeFooter;

    @Value("${event.coversheet.spacing.betweenparagraph}")
    private String betweenParagraph;

    @Value("${event.coversheet.font.keysize}")
    private String keySize;

    @Value("${event.coversheet.font.valuesize}")
    private String valueSize;

    @Value("${event.coversheet.font.keyfont}")
    private String keyFontProp;

    @Value("${event.coversheet.font.valuefont}")
    private String valueFontProp;

    @Value("${event.coversheet.alignment.left}")
    private String alignmentLeft;

    @Value("${event.coversheet.alignment.right}")
    private String alignmentRight;

    @Value("${event.coversheet.alignment.bottom}")
    private String alignmentBottom;

    @Autowired
    private SpecialInstructionsRepository specialInstructionsRepository;

    private class HeaderFooter extends PdfPageEventHelper {
        private RequestEvent requestEvent;
        private Font keyFont;
        private Font valueFont;

        public HeaderFooter(RequestEvent requestEvent, Font keyFont, Font valueFont) {
            this.requestEvent = requestEvent;
            this.keyFont = keyFont;
            this.valueFont = valueFont;
        }

        @Override
        public void onEndPage(PdfWriter writer, Document document) {
            PdfContentByte cb = writer.getDirectContent();
            float rightX = document.right();
            float headerY = document.getPageSize().getHeight() - 30;
            float lineSpacing = 22f;

            drawRightAlignedMixedText(cb, UMFR, maskText(getOrDefault(requestEvent.getUmfr()), 38), keyFont, valueFont, rightX, headerY - lineSpacing);
            drawRightAlignedMixedText(cb, EVENTID, maskText(getOrDefault(requestEvent.getId()), 38), keyFont, valueFont, rightX, headerY - 2 * lineSpacing);
            drawRightAlignedMixedText(cb, CUSTOMER_ID, maskText(getOrDefault(requestEvent.getCustMstNo()), 30), keyFont, valueFont, rightX, headerY - 3 * lineSpacing);
            drawRightAlignedMixedText(cb, CUSTOMER_NAME, maskText(getOrDefault(requestEvent.getCusName()), 30), keyFont, valueFont, rightX, headerY - 4 * lineSpacing);
        }

        private void drawRightAlignedMixedText(PdfContentByte cb, String label, String value, Font keyFont, Font valueFont, float rightX, float y) {
            try {
                float labelWidth = keyFont.getBaseFont().getWidthPoint(label, keyFont.getSize());
                float valueWidth = valueFont.getBaseFont().getWidthPoint(value, valueFont.getSize());
                float totalWidth = labelWidth + valueWidth;
                float startX = rightX - totalWidth;

                cb.beginText();
                cb.setFontAndSize(keyFont.getBaseFont(), keyFont.getSize());
                cb.setTextMatrix(startX, y);
                cb.showText(label);
                cb.endText();

                cb.beginText();
                cb.setFontAndSize(valueFont.getBaseFont(), valueFont.getSize());
                cb.setTextMatrix(startX + labelWidth, y);
                cb.showText(value);
                cb.endText();
            } catch (Exception e) {
                // Handle font exceptions gracefully
                System.err.println("Error drawing text: " + e.getMessage());
            }
        }

        private String getOrDefault(String str) {
            return (str == null || str.trim().isEmpty()) ? "-" : str;
        }

        private String maskText(String text, int maxLength) {
            if (text.length() > maxLength) {
                return text.substring(0, maxLength - 3) + "...";
            }
            return text;
        }
    }

    public byte[] export(RequestEvent requestEvent) throws DocumentException, IOException {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        Document document = new Document(PageSize.A4);
        document.setMargins(36, 36, 140, 35);

        PdfWriter writer = PdfWriter.getInstance(document, byteArrayOutputStream);

        Font keyFont = FontFactory.getFont(keyFontProp);
        keyFont.setSize(Float.parseFloat(keySize));
        Font valueFont = FontFactory.getFont(valueFontProp);
        valueFont.setSize(Float.parseFloat(valueSize));

        writer.setPageEvent(new HeaderFooter(requestEvent, keyFont, valueFont));

        document.open();

        // Product Group
        Paragraph e5 = new Paragraph();
        Chunk productGroupKey = new Chunk(PRODUCT_GROUP, keyFont);
        Chunk productGroupValue = new Chunk(Objects.toString(requestEvent.getProductGroupName(), ""), valueFont);
        e5.add(productGroupKey);
        e5.add(productGroupValue);
        e5.setSpacingAfter(Float.valueOf(betweenProperties).floatValue());
        e5.setAlignment(Integer.parseInt(alignmentLeft));
        document.add(e5);

        // Product
        Paragraph e6 = new Paragraph();
        Chunk productKey = new Chunk(PRODUCT, keyFont);
        Chunk productValue = new Chunk(Objects.toString(requestEvent.getProductName(), ""), valueFont);
        e6.add(productKey);
        e6.add(productValue);
        e6.setSpacingAfter(Float.valueOf(betweenProperties).floatValue());
        e6.setAlignment(Integer.parseInt(alignmentLeft));
        document.add(e6);

        // Event
        Paragraph e7 = new Paragraph();
        Chunk eventKey = new Chunk(EVENT, keyFont);
        Chunk eventValue = new Chunk(Objects.toString(requestEvent.getEventName(), ""), valueFont);
        e7.add(eventKey);
        e7.add(eventValue);
        e7.setAlignment(Integer.parseInt(alignmentLeft));
        e7.setSpacingAfter(Float.valueOf(betweenProperties).floatValue());
        document.add(e7);

        // Event Priority
        Paragraph e8 = new Paragraph();
        Chunk eventPriorityKey = new Chunk(EVENT_PRIORITY, keyFont);
        Chunk eventPriorityValue = new Chunk(Objects.toString(requestEvent.getEventPriorityName(), ""), valueFont);
        e8.add(eventPriorityKey);
        e8.add(eventPriorityValue);
        e8.setAlignment(Integer.parseInt(alignmentLeft));
        e8.setSpacingAfter(Float.valueOf(betweenProperties).floatValue());
        document.add(e8);

        // Event Initiation
        String initiationDate = dateFormatter(requestEvent.getSystemCreationDate());
        Paragraph e9 = new Paragraph();
        Chunk eventInitiationKey = new Chunk(EVENT_INITIATION, keyFont);
        Chunk eventInitiationValue = new Chunk(initiationDate != null ? initiationDate + " (UTC)" : "", valueFont);
        e9.add(eventInitiationKey);
        e9.add(eventInitiationValue);
        e9.setAlignment(Integer.parseInt(alignmentLeft));
        e9.setSpacingAfter(Float.valueOf(betweenProperties).floatValue());
        document.add(e9);

        // Special Instructions
        List<SpecialInstructionsEntity> instructions = specialInstructionsRepository.findByEventIdOrderByCreatedOnDesc(requestEvent.getId());
        if (instructions != null && !instructions.isEmpty()) {
            Paragraph headerParagraph = new Paragraph(SPECIAL_INSTRUCTION, valueFont);
            headerParagraph.setAlignment(Element.ALIGN_CENTER);
            headerParagraph.setSpacingAfter(5f);
            document.add(headerParagraph);

            createImprovedTwoColumnInstructions(document, writer, instructions, keyFont);
        }

        document.close();
        return byteArrayOutputStream.toByteArray();
    }

    private void createImprovedTwoColumnInstructions(Document document, PdfWriter writer, List<SpecialInstructionsEntity> instructions, Font font) {
        float pageWidth = document.getPageSize().getWidth();
        float pageHeight = document.getPageSize().getHeight();
        float left = document.leftMargin();
        float right = document.rightMargin();
        float top = document.topMargin();
        float bottom = document.bottomMargin() + 20;
        float COLUMN_GAP = 15;

        float usableWidth = pageWidth - left - right;
        float columnWidth = (usableWidth - COLUMN_GAP) / 2;

        // Start from current position
        float yStart = pageHeight - top - 80;
        
        try {
            float currentY = writer.getVerticalPosition(true);
            if (currentY > 0 && currentY < yStart) {
                yStart = currentY - 10;
            }
        } catch (Exception e) {
            // Use default yStart if unable to get current position
        }

        float yEnd = bottom;

        // Build all instruction paragraphs
        List<InstructionItem> instructionItems = buildInstructionItems(instructions, font);
        
        // Use smart column distribution
        distributeInstructionsAcrossColumns(document, writer, instructionItems, left, columnWidth, COLUMN_GAP, yStart, yEnd, font);
    }

    private static class InstructionItem {
        String headerText;
        List<String> paragraphTexts;
        
        InstructionItem(String headerText, List<String> paragraphTexts) {
            this.headerText = headerText;
            this.paragraphTexts = paragraphTexts;
        }
    }

    private List<InstructionItem> buildInstructionItems(List<SpecialInstructionsEntity> instructions, Font font) {
        List<InstructionItem> items = new ArrayList<>();
        int count = 1;

        for (SpecialInstructionsEntity instruction : instructions) {
            String createdOn = instruction.getCreatedOn() != null ? dateFormatter(instruction.getCreatedOn()) : "";
            String createdBy = instruction.getCreatedByUser() != null ? instruction.getCreatedByUser() : "";
            String comment = instruction.getComment() != null ? cleanHtml(instruction.getComment()) : "";
            
            // Truncate if needed
            comment = truncateToDbLimit(comment, 32500);
            
            String headerText = String.format("[%s] [%s]\n%d) ", createdOn, createdBy, count);
            
            // Split comment into natural paragraphs
            List<String> paragraphs = new ArrayList<>();
            String[] paragraphsInComment = comment.split("\\n|\\r\\n|\\r|<br>|<br/>|</p>\\s*<p>");
            
            for (String paragraphText : paragraphsInComment) {
                paragraphText = paragraphText.trim();
                if (!paragraphText.isEmpty()) {
                    paragraphs.add(paragraphText);
                }
            }
            
            if (paragraphs.isEmpty()) {
                paragraphs.add(""); // Ensure at least one paragraph
            }
            
            items.add(new InstructionItem(headerText, paragraphs));
            count++;
        }

        return items;
    }

    private void distributeInstructionsAcrossColumns(Document document, PdfWriter writer, List<InstructionItem> items, 
                                                   float left, float columnWidth, float columnGap, float yStart, float yEnd, Font font) {
        
        float pageHeight = document.getPageSize().getHeight();
        float pageTop = document.topMargin();
        
        ColumnText leftColumn = new ColumnText(writer.getDirectContent());
        ColumnText rightColumn = new ColumnText(writer.getDirectContent());
        
        // Set up columns
        leftColumn.setSimpleColumn(left, yEnd, left + columnWidth, yStart);
        rightColumn.setSimpleColumn(left + columnWidth + columnGap, yEnd, left + 2 * columnWidth + columnGap, yStart);
        
        leftColumn.setLeading(font.getSize() * 1.3f);
        rightColumn.setLeading(font.getSize() * 1.3f);
        
        int currentItemIndex = 0;
        boolean useLeftColumn = true;
        
        while (currentItemIndex < items.size()) {
            InstructionItem item = items.get(currentItemIndex);
            
            // Create paragraphs for this item
            List<Paragraph> itemParagraphs = createParagraphsForItem(item, font);
            
            // Try to fit the complete item in current column
            ColumnText targetColumn = useLeftColumn ? leftColumn : rightColumn;
            ColumnText testColumn = ColumnText.duplicate(targetColumn);
            
            // Test if entire item fits
            boolean itemFits = true;
            for (Paragraph p : itemParagraphs) {
                testColumn.addElement(p);
            }
            
            int status = testColumn.go(true); // Simulate
            if (ColumnText.hasMoreText(status)) {
                itemFits = false;
            }
            
            if (itemFits) {
                // Item fits, add it to the target column
                for (Paragraph p : itemParagraphs) {
                    targetColumn.addElement(p);
                }
                currentItemIndex++;
                
                // Switch to other column for next item (for better distribution)
                useLeftColumn = !useLeftColumn;
            } else {
                // Item doesn't fit in current column
                if (useLeftColumn) {
                    // Try right column
                    useLeftColumn = false;
                    continue;
                } else {
                    // Neither column has space, render current page and start new one
                    leftColumn.go();
                    rightColumn.go();
                    
                    // Start new page
                    document.newPage();
                    float newYStart = pageHeight - pageTop - 40;
                    
                    leftColumn = new ColumnText(writer.getDirectContent());
                    rightColumn = new ColumnText(writer.getDirectContent());
                    
                    leftColumn.setSimpleColumn(left, yEnd, left + columnWidth, newYStart);
                    rightColumn.setSimpleColumn(left + columnWidth + columnGap, yEnd, left + 2 * columnWidth + columnGap, newYStart);
                    
                    leftColumn.setLeading(font.getSize() * 1.3f);
                    rightColumn.setLeading(font.getSize() * 1.3f);
                    
                    useLeftColumn = true; // Start with left column on new page
                }
            }
        }
        
        // Render remaining content
        leftColumn.go();
        rightColumn.go();
    }

    private List<Paragraph> createParagraphsForItem(InstructionItem item, Font font) {
        List<Paragraph> paragraphs = new ArrayList<>();
        
        boolean isFirstParagraph = true;
        for (String paragraphText : item.paragraphTexts) {
            String fullText;
            if (isFirstParagraph) {
                fullText = item.headerText + paragraphText;
                isFirstParagraph = false;
            } else {
                fullText = "   " + paragraphText; // Indent continuation paragraphs
            }
            
            // Handle very long paragraphs by intelligent line breaking
            if (fullText.length() > 800) {
                List<String> brokenText = intelligentLineBreak(fullText, 800);
                for (String text : brokenText) {
                    Paragraph p = new Paragraph(text, font);
                    p.setSpacingAfter(4f);
                    p.setIndentationLeft(3f);
                    p.setAlignment(Element.ALIGN_JUSTIFIED);
                    paragraphs.add(p);
                }
            } else {
                Paragraph p = new Paragraph(fullText, font);
                p.setSpacingAfter(6f);
                p.setIndentationLeft(3f);
                p.setAlignment(Element.ALIGN_JUSTIFIED);
                paragraphs.add(p);
            }
        }
        
        // Add extra spacing after complete item
        if (!paragraphs.isEmpty()) {
            paragraphs.get(paragraphs.size() - 1).setSpacingAfter(12f);
        }
        
        return paragraphs;
    }

    private List<String> intelligentLineBreak(String text, int maxLength) {
        List<String> result = new ArrayList<>();
        
        if (text == null || text.isEmpty()) {
            return result;
        }
        
        String[] sentences = text.split("\\. ");
        StringBuilder currentChunk = new StringBuilder();
        
        for (int i = 0; i < sentences.length; i++) {
            String sentence = sentences[i];
            if (i < sentences.length - 1 && !sentence.endsWith(".")) {
                sentence += "."; // Re-add the period
            }
            
            if (currentChunk.length() + sentence.length() + 1 <= maxLength) {
                if (currentChunk.length() > 0) {
                    currentChunk.append(" ");
                }
                currentChunk.append(sentence);
            } else {
                if (currentChunk.length() > 0) {
                    result.add(currentChunk.toString());
                    currentChunk = new StringBuilder();
                }
                
                // Handle very long sentences
                if (sentence.length() > maxLength) {
                    result.addAll(splitLongSentence(sentence, maxLength));
                } else {
                    currentChunk.append(sentence);
                }
            }
        }
        
        if (currentChunk.length() > 0) {
            result.add(currentChunk.toString());
        }
        
        return result;
    }

    private List<String> splitLongSentence(String sentence, int maxLength) {
        List<String> result = new ArrayList<>();
        
        while (sentence.length() > maxLength) {
            int splitPoint = maxLength;
            
            // Try to find a good break point (space, comma, etc.)
            for (int i = maxLength - 1; i > maxLength - 50 && i > 0; i--) {
                char c = sentence.charAt(i);
                if (c == ' ' || c == ',' || c == ';') {
                    splitPoint = i + 1;
                    break;
                }
            }
            
            result.add(sentence.substring(0, splitPoint).trim());
            sentence = sentence.substring(splitPoint).trim();
        }
        
        if (!sentence.isEmpty()) {
            result.add(sentence);
        }
        
        return result;
    }

    private String truncateToDbLimit(String text, int maxLength) {
        if (text == null) {
            return "";
        }
        
        if (text.length() <= maxLength) {
            return text;
        }
        
        // Try to truncate at a sentence boundary
        String truncated = text.substring(0, maxLength - 3);
        int lastPeriod = truncated.lastIndexOf('.');
        
        if (lastPeriod > maxLength - 200) { // Only use sentence boundary if it's close to the limit
            truncated = truncated.substring(0, lastPeriod + 1);
        } else {
            // Fallback to word boundary
            int lastSpace = truncated.lastIndexOf(' ');
            if (lastSpace > maxLength - 100) {
                truncated = truncated.substring(0, lastSpace);
            }
        }
        
        return truncated + "...";
    }

    private String cleanHtml(String input) {
        if (input == null) {
            return "";
        }
        return input.replaceAll("<[^>]*>", "")
                   .replaceAll("&nbsp;", " ")
                   .replaceAll("&amp;", "&")
                   .replaceAll("&lt;", "<")
                   .replaceAll("&gt;", ">")
                   .replaceAll("&quot;", "\"")
                   .trim();
    }

    private String dateFormatter(LocalDateTime dt) {
        if (dt == null) {
            return "";
        }
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("d'th' MMMM yyyy HH:mm");
        return dt.format(formatter);
    }
}
