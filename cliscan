#!/usr/bin/python3
'''
Simple helper script to scan documents via the console
'''
import logging
from argparse import ArgumentParser
from os import listdir, unlink
from os.path import join
from subprocess import check_call, check_output
from tempfile import NamedTemporaryFile, TemporaryDirectory
from typing import BinaryIO, List

LOG = logging.getLogger(__name__)

# Modes
#    - Lineart
#    - Gray
#    - Color
MODE = 'Gray'

# Resolutions
#     - 75
#     - 100
#     - 150
#     - 300
#     - 600
#     - 1200
RESOLUTION = 150

# Scan Areas (width, height) for batch scanning
A4 = (210, 297)


def topdf(files: List[BinaryIO], outfile: BinaryIO):
    '''
    Convert files to PDF
    '''
    filenames = [fle.name for fle in files]
    LOG.debug('Converting %r to %r', filenames, outfile.name)
    cmd = ['convert']
    cmd.extend(filenames)
    cmd.append(outfile.name)
    check_call(cmd)


def scan_adf(adf_folder: str = ''):
    '''
    Scan multiple pages page into a temporary folder
    '''
    dimension = A4
    cmd = [
        'scanimage',
        '--format=tiff',
        '--source', 'Automatic Document Feeder',
        '--progress',
        '--mode=%s' % MODE,
        '--resolution=%ddpi' % RESOLUTION,
        '--batch=%s/page-%%04d.tiff' % adf_folder,
        '-l', '0.0',
        '-t', '0.0',
        '-x', dimension[0],
        '-y', dimension[1],
    ]
    LOG.debug('Reading ADF scan into %s...', adf_folder)
    check_call(cmd)


def scan(outfile: BinaryIO):
    '''
    Scan one page infor a file object

    :param oufile: The (binary) file pointer into which the file will be saved.
    '''
    cmd = [
        'scanimage',
        '--format=tiff',
        '--progress',
        '--mode=%s' % MODE,
        '--resolution=%ddpi' % RESOLUTION,
    ]
    LOG.debug('Reading scan into memory...')
    data = check_output(cmd)
    LOG.debug('Writing into %r', outfile.name)
    outfile.write(data)


def shrink(infile: BinaryIO, outfile: BinaryIO, dpi: int = 150) -> None:
    '''
    Shrinks a big PDF file by downsizing images
    '''
    LOG.debug('Shrinking %r to %r with image DPI %r',
              infile.name, outfile.name, dpi)
    cmd = [
        'gs',
        '-q',
        '-dNOPAUSE',
        '-dBATCH',
        '-dSAFER',
        '-sDEVICE=pdfwrite',
        '-dCompatibilityLevel=1.3',
        '-dPDFSETTINGS=/screen',
        '-dEmbedAllFonts=true',
        '-dSubsetFonts=true',
        '-dColorImageDownsampleType=/Bicubic',
        '-dColorImageResolution=%d' % dpi,
        '-dGrayImageDownsampleType=/Bicubic',
        '-dGrayImageResolution=%d' % dpi,
        '-dMonoImageDownsampleType=/Bicubic',
        '-dMonoImageResolution=%d' % dpi,
        '-sOutputFile=%s' % outfile.name,
        infile.name,
    ]
    check_call(cmd)


def onepage(outfile: str = 'output.pdf'):
    '''
    Scan one page as PDF document
    '''
    with NamedTemporaryFile(suffix='.tiff') as tmpimg, \
            NamedTemporaryFile(suffix='.pdf') as tmppdf, \
            open(outfile, 'wb') as fptr:
        scan(tmpimg)
        topdf([tmpimg], tmppdf)
        shrink(tmppdf, fptr)


def multipage(outfile: str = 'output.pdf'):
    '''
    Scan multiple pages as PDF document
    '''
    do_continue = True
    files = []

    try:
        while do_continue:
            with NamedTemporaryFile(suffix='.tiff', delete=False) as tmpimg:
                scan(tmpimg)
                files.append(tmpimg)
            do_continue = input('To finish, type any text: ').strip() == ''

        with NamedTemporaryFile(suffix='.pdf') as tmppdf, \
                open(outfile, 'wb') as fptr:
            topdf(files, tmppdf)
            shrink(tmppdf, fptr)
    finally:
        for fle in files:
            unlink(fle.name)


def multi_adf(outfile: str = 'output.pdf'):
    '''
    Scan multiple pages as PDF document from the automatic document feeder
    '''
    with TemporaryDirectory() as tmpdir:
        scan_adf(tmpdir)
        files = [open(join(tmpdir, fname), 'rb')
                 for fname in sorted(listdir(tmpdir))
                 if not fname.startswith('.')]
        try:
            with NamedTemporaryFile(suffix='.pdf') as tmppdf, \
                    open(outfile, 'wb') as fptr:
                topdf(files, tmppdf)
                shrink(tmppdf, fptr)
        finally:
            for fle in files:
                fle.close()


def parse_args():
    '''
    Parse CLI argumetns
    '''
    parser = ArgumentParser()
    parser.add_argument('-m', '--multipage', action='store_true',
                        default=False)
    parser.add_argument('--adf', action='store_true', default=False)
    parser.add_argument('outfile', help='output file (PDF)')
    args = parser.parse_args()
    if not args.outfile.lower().endswith('.pdf'):
        parser.error('Output filenames must end with .pdf!')
    return args


def main():
    '''
    Main entry point
    '''
    args = parse_args()
    logging.basicConfig(level=logging.DEBUG)
    if args.adf:
        multi_adf(args.outfile)
    elif args.multipage:
        multipage(args.outfile)
    else:
        onepage(args.outfile)


if __name__ == '__main__':
    main()
