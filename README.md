# productivity-checklist

A self-contained static productivity checklist web app (single `index.html`, no build step).

**Live site:** https://productivity-checklist.vercel.app

## Deploying

This repo is connected to Vercel. Every push to the `main` branch automatically
redeploys the live site — just edit `index.html`, then:

```bash
git add index.html
git commit -m "Update checklist"
git push
```

The production URL updates on its own within seconds.

<!-- Auto-deploy connected and verified. -->

## מסלולי הליכה בתל אביב (`walking-routes.html`)

אפליקציית ווב עצמאית (קובץ HTML יחיד, ללא build) למסלולי הליכה מעגליים בתל אביב,
בסגנון מפת פיקסל ארט רטרו 16 ביט, עם נקודת בית קבועה ("הבית שלי", החלוצים 48).

- ממשק מלא בעברית RTL.
- מבוססת **Google Maps JavaScript API + Directions API** לנתוני הליכה אמיתיים.
- עובדת גם **ללא מפתח API** במצב דמו (מפת פיקסל ארט מקומית, מסלולים מחושבים מקומית).
- שלוש תצוגות: פיקסל ארט / משולב / Google רגיל.

האופי הוויזואלי של כל שכונה (פלורנטין, נווה צדק, יפו, פארק המסילה, שוק הכרמל, רוטשילד,
הבימה, הטיילת ועוד) נקבע לפי **מחקר אמיתי** על כל אזור — לא לפי הנחות מהשם.

**הפעלה:** פותחים את `walking-routes.html` בדפדפן. לחיבור נתונים אמיתיים מחליפים בקוד
את `PASTE_GOOGLE_MAPS_API_KEY_HERE` במפתח Google Maps (Maps JavaScript API + Directions API).

